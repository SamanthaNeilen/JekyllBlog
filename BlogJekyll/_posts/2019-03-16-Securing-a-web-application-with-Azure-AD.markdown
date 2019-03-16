---
layout: post
title:  "Getting started with web application security using Azure Active Directory"
date:   2019-03-16 00:00:00 +0100

tags: WebAPI, Mvc
---

When developing enterprise applications on the internet you want to limit access to authenticated users only. In this blogpost I will provide links to some guides to help you get started with implementing secured websites and API using Azure Active Directory.

For more information on how the authentication methods provided by Azure Active Directory see the [Microsoft Docs pages for Azure Active Directory authentication.](https://docs.microsoft.com/en-us/azure/active-directory/develop/authentication-scenarios)

Also a great comprehensive resource around Azure Active Directory security is the ["Azure Active Directory for Developers" Pluralsight course](https://app.pluralsight.com/library/courses/azure-active-directory-developers/table-of-contents).

### Azure Active Directory security for a user using a browser (OpenID Connect)

When using Azure Active Directory for a user, the actual authentication process will happen outside of your application via the [OpenID Connect authentication scheme](https://openid.net/connect/). When an unauthenticated user uses a browser to navigate to your application, the middleware code will redirect the user to a Microsoft hosted login page. When the user provides the correct credentials, the login page will redirect the authenticated user back to a specified redirect URL in your application.

There are already great guides and sample application from Microsoft on how to get started with Azure Active Directory in your web applications.

For a guide on setting up OpenID Connect authentication for a .NET Framework application see the  ["Add sign-in with Microsoft to an ASP.NET web app" guide on the Microsoft Docs pages.](https://docs.microsoft.com/en-us/azure/active-directory/develop/tutorial-v2-asp-webapp ) or the ["Integrate Azure AD into a web application using OpenID Connect" Azure examples article.](https://azure.microsoft.com/en-us/resources/samples/active-directory-dotnet-webapp-openidconnect/)

For .NET Core see the ["Azure Active Directory with ASP.NET Core" Microsoft Docs page](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/azure-active-directory/) or the ["Integrating Azure AD into an ASP.NET Core web app" Azure examples article](https://azure.microsoft.com/en-us/resources/samples/active-directory-dotnet-webapp-openidconnect-aspnetcore/).

When using UI tests on a website that is secured with OpenID Connect you will also have to automate the login process. You will have to verify if a user lands on a login page and then use the automation framework to fill in a valid user name and password.

### Azure Active Directory security between applications (Bearer token authentication) 

When another applications requests or posts data to your API, you will want to make sure that the API facing the public internet is secured so that only no unauthorized parties can use the API.

This security mechanism can leverage the Azure Active Directory via so called [access token](https://docs.microsoft.com/en-us/azure/active-directory/develop/access-tokens). The requesting party can request a token and send it in the Authorization header of the request to the API. The API application can verify the validity of the token against Azure Active Directory. If a token is valid the API can process the request and can use the caller identity and claims from the token available for further authorization logic.

For a guide on setting up bearer token authentication for a .NET Framework API application see the ["Build a .NET web API that integrates with Azure AD for authentication and authorization" quickstart on the Microsoft Docs pages.](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-v1-dotnet-webapi)

For a guide on setting up bearer token authentication for a .NET Core API application see the ["JWT Validation and Authorization in ASP.NET Core" guide on the Microsoft Docs pages](https://devblogs.microsoft.com/aspnet/jwt-validation-and-authorization-in-asp-net-core/). 

If the client application that consumes  the API is a .NET application, it will probably use the [ADAL](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki) or [MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki) client libraries to request a valid bearer token for the API. MSAL is currently still in preview. 

If the current logged in user of the client application does not matter for the data retrieved or sent to the API, you can request a token for the client application without including any indication of the user that is logged into the client application. This will allow applications to be able to communicate securely without knowing about each others users or the privileges that the users have within a client application. This is called the "client credential" OAuth grant/flow. For more information on the flows/grants available in OAuth see the article ["Which OAuth flow to use"](https://auth0.com/docs/api-auth/which-oauth-flow-to-use).

For a guide on requesting a client credential flow token using ADAL see the ["client credential flow" Github wiki page](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Client-credential-flows).

For a guide on requesting a client credential flow token using MSAL see the ["client credential flow" Github wiki page](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Client-credential-flows).

If the client application is a non .NET application, it can request a token for the client application using the OAuth endpoints of Azure Active Directory via HTTP posts. See [the Microsoft docs pages for the HTTP requests to request an access token for an application](https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-oauth2-client-creds-grant-flow#service-to-service-access-token-request).

For testing an API with automated tests you can request a bearer token for the API with the same code as a client application and include the token in the Authorization header when calling the API under test.

### Global filters to ensure all requests need to have an authenticated user

After setting up authentication policies you will have to ensure that all requests require an authenticated user. This can be done by using the Authorize attribute on all (base) controllers but there are also global filters for .NET Framework or Authentication policies for .NET Core. 

#### Global filters to ensure all requests need to have an authenticated user (.NET Framework)

To set up a global Authorization attribute in .NET Framework add the code below to the App_start/FilterConfig.cs file:

```
public class FilterConfig
{
    public static void RegisterGlobalFilters(GlobalFilterCollection filters)
    {
        filters.Add(new AuthorizeAttribute());
    }
}
```
The Authorize attributes for MVC and API Controllers reside in a different namespaces. If you decorate and API controller with the Authorize attribute for a MVC controller, the Authorize attribute will not be triggered. The MVC authorize attribute uses the "System.Web.Mvc" namespace. The API authorize attribute uses the "System.Web.Http" namespace.

If you have a project with both MVC and Web API controllers. You may want to configure the default OWIN Authentication pipeline to accept both OpenIDConnect and JwtBearerTokens but have certain API Controllers only use the JwtBearerToken authentication. You can deviate from the default OWIN Configuration by using the HostAuthenticationAttribute (System.Web.Http) on a controller or the SuppressHostPrincipal in the App_Start\WebApiConfig.cs file. You will need to install the NuGet package: "Microsoft.AspNet.WebApi.Owin" to access the classes.

Example for the global filter in the App_Start\WebApiConfig.cs:
```
config.SuppressDefaultHostAuthentication();
config.Filters.Add(new HostAuthenticationFilter("Bearer"));
```

Example for the attribute on a specific controller::
```
[HostAuthentication("Bearer")]
[Authorize]
public class MyApiController 
```

#### Global filters to ensure all requests need to have an authenticated user (.NET Core)

To enable a global filters for all API and MVC controllers in .NET Core  you can use a Convention to set up an AuthorizationFilter on all controllers.

To implement the Convention, first add a new IControllerModelConvention implementation that will set the authorization policy for the controller. You could add extra conditional logic to not set the policy under certain conditions.

```
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc.ApplicationModels;
using Microsoft.AspNetCore.Mvc.Authorization;

namespace NetCore.WebApiExample
{
    public class GlobalAuthorizeControllerModelConvention : IControllerModelConvention
    {
        public void Apply(ControllerModel controller)
        {
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .Build();

            controller.Filters.Add(new AuthorizeFilter(policy));
        }
    }
}
```
Next apply the convention in the Startup.cs file:

``` 
public void ConfigureServices(IServiceCollection services)
{
    //other code omitted
    //...

    services.AddMvc(options =>
    {
        options.Conventions.Add(new GlobalAuthorizeControllerModelConvention());
    }).SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
```

For more fine-grained authentication logic you could also implement a IActionModelConvention to conditionally apply the authorization policy per controller action.

To enable authentication for specific Razor Pages see [the Microsoft Docs pages on Razor Pages authorization conventions](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/razor-pages-authorization).
