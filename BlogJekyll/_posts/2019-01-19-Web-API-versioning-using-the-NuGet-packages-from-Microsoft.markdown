---
layout: post
title:  "Web API versioning using the NuGet packages from Microsoft"
date:   2019-01-19 00:00:00 +0100

tags: WebAPI
---

When developing a Web API, using versioning can greatly improve flexibility for production deployments. Even if the contract for your API changes, current users will not be impacted if you keep supporting the version they currently use. Using versioning will allow the consumers of your service more time to upgrade to newer versions and it will at the same time allow you to deploy on demand. At some point in time certain versions should of course be deprecated and finally removed to improve maintainability of the software.  

Microsoft has several NuGet packages available to easily configure versioning in your API: 

**NET core:** <br/>
Microsoft.AspNetCore.Mvc.Versioning<br/>
Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer

**NET framework:**<br/>
Microsoft.AspNet.WebApi.Versioning<br/>
Microsoft.AspNet. WebApi.Versioning.ApiExplorer

The ApiExplorer package is only needed when using the API metadata for each version (like for Swagger documentation). 

In this blogpost I will describe how to enable versioning for your API including version support for the Swagger UI documentation. This blogpost will only describe these steps for a .NET Core projects. A complete solution with the code can be found in my [WebApiExamples GitHub repository](https://github.com/SamanthaNeilen/WebApiExamples " WebApiExamples GitHub repository "). A .NET Framework example can be found on the [Microsoft.Asp.Net.WebApi.Versioning github repository for a sample project](https://github.com/Microsoft/aspnet-api-versioning/tree/master/samples/webapi/SwaggerWebApiSample "Microsoft.Asp.Net.WebApi.Versioning github repository for a sample project"). The default Startup classes for the .NET Framework will need to be set up a little different from a project without versioning.  

A small disclaimer: I will only show how to get started with versioning. When creating a new version, you should always think about support for previous versions or graceful degradation. Especially when you do database or schema changes. You must be careful not to break a previous version and may have to support multiple classes (or even database objects) with v1, v2 etcetera in their name or namespace if they are still used in a previous supported version. It is always a good idea when building an API to have integration tests. Having regression tests for the previous versions can ensure not breaking those versions while focusing on the new version.

The [Microsoft.Asp.Net.WebApi.Versioning github repository wiki](https://github.com/Microsoft/aspnet-api-versioning/wiki "Microsoft.Asp.Net.WebApi.Versioning github repository wiki") contains all the documentation for the configuration and possibilities of the Microsoft.Asp.Net.WebApi.Versioning packages.

### Minimal configuration for versioning

First install the NuGet package Microsoft.AspNetCore.Mvc.Versioning in your web project. 

Next go to the Startup.cs file and add the code shown below to enable the default versioning by querystring. 

```c#
public void ConfigureServices(IServiceCollection services)
{
    //other code omitted
    services.AddApiVersioning(o =>
    {
        o.AssumeDefaultVersionWhenUnspecified = true;
        o.DefaultApiVersion = new ApiVersion(1, 0);
    });
    
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
```

This is all you need to enable versioning. The 2 options set in this snippet ensure that your current API will act as version 1.0 and that everyone that calls your service without a version querystring receives that version. (If you use Swagger documentation via Swashbuckle you will need extra configuration to ensure Swagger file generation keeps working. See the "Setup versioning for Swagger documentation using the Swashbuckle NuGet package" section later in this post.) 

In my [WebApiExamples GitHub repository](https://github.com/SamanthaNeilen/WebApiExamples " WebApiExamples GitHub repository ") I have an API endpoint on the route <span class="link-style">https://localhost:44314/api/routingapi/123</span>. After adding the versioning configuration, this API is still available and responding to that address. Next I can call <span class="link-style">https://localhost:44314/api/routingapi/123?api-version=2.0</span>. Since I did not define a version 2.0 (at this point), I will receive the automatic error generated from the versioning package as shown below. 

```json
{"error":{"code":"UnsupportedApiVersion","message":"The HTTP resource that matches the request URI 'https://localhost:44314/api/routingapi/123' does not support the API version '2.0'.","innerError":null}}
```

I can now add a RouteApiV2Controller with a new interface to listen to the same url on V2. The code for the RouteApiController (that’s listening to version 1.0 or no ApiVersion specification) and RouteApiV2Controller are shown below.

 RouteApiController:

```c#
using Microsoft.AspNetCore.Mvc;

namespace NetCore.WebApiExample.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    [ApiVersion("1.0")]
    public class RoutingApiController : ControllerBase
    {
        [HttpGet]
        [Route("{email:Email}")]
        public IActionResult Get(string email)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(email)}) (DEFAULT VERSION)");
        }        

        [HttpGet]
        [Route("{validPositiveInt32Value:range(0,2147483647)}")]
        public IActionResult Get(int validPositiveInt32Value)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(validPositiveInt32Value)} (DEFAULT VERSION)");
        }

        [HttpGet]
        [Route("{inputMatchingRegex:regex(^(.+)(_)(.+)$)}")]
        public IActionResult GetByRegex(string inputMatchingtRegexForValueContainingUnderscore)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(inputMatchingtRegexForValueContainingUnderscore)}) (DEFAULT VERSION)");
        }
    }
}
```

RouteApiV2Controller:

```c#
using Microsoft.AspNetCore.Mvc;

namespace NetCore.WebApiExample.Controllers
{
    [Route("api/RoutingApi")]
    [ApiController]
    [ApiVersion("2.0")]    
    public class RoutingApiV2Controller : ControllerBase
    {
        [HttpGet]
        [Route("{email:Email}")]
        public IActionResult Get(string email)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(email)}) (V2)");
        }

        [HttpGet]
        [Route("{validPositiveInt32Value:range(0,2147483647)}")]
        public IActionResult Get(int validPositiveInt32Value)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(validPositiveInt32Value)} (V2)");
        }
    }
}
```

Notice that the V2 controller maps to the same path but is missing an endpoint. 

When calling <span class="link-style">https://localhost:44314/api/routingapi/123</span> the service will return the default version. When calling <span class="link-style">https://localhost:44314/api/routingapi/123?api-version=2.0</span> it will execute and return the RouteApiV2Controller version. 

When calling the missing method in v2 <span class="link-style">https://localhost:44314/api/routingapi/some_value?api-version=2.0</span> it will return the error shown below. When calling <span class="link-style">https://localhost:44314/api/routingapi/some_value</span> or <span class="link-style">https://localhost:44314/api/routingapi/some_value?api-version=1.0</span> it will just return the old method for version 1.

```json
{
    "error": {
        "code": "UnsupportedApiVersion",
        "message": "The HTTP resource that matches the request URI 
            'https://localhost:44314/api/routingapi/some_value' does not support the API version '2.0'.",
        "innerError": null
    }
}
```

### Setup versioning for Swagger documentation using the Swashbuckle NuGet package

If you use Swagger UI via the Swashbuckle package as online documentation for your API you can set up descriptions per version. This will allow you to write release notes per versions and will give consumers documentation that represents actual the version the API that they currently use even if newer versions are available.

If you are using Swagger, the generation of the Swagger file will fail if routes are available for multiple versions and you have changed the configuration for Swagger to support multiple versions.

First install the Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer NuGet package.

Next configure the Swagger configuration for versions in the ConfigureService method in the StartUp.cs as shown below to change the generation of a Swagger.json file per version.  

```c#
public void ConfigureServices(IServiceCollection services)
{
    // other code omitted 	

    // ConfigureSwagger(services) contains the services.AddSwaggerGen(options => {...}) code see method definition below
    ConfigureSwagger(services);
 
    services.AddApiVersioning(o =>
        {
	        o.AssumeDefaultVersionWhenUnspecified = true;
	        o.DefaultApiVersion = new ApiVersion(1, 0);
        });

    services.AddVersionedApiExplorer(o => o.GroupNameFormat = "'V'VVV");
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}

private void ConfigureSwagger(IServiceCollection services)
{
    services.AddSwaggerGen(options =>
    {
        var provider = services.BuildServiceProvider()
            .GetRequiredService<IApiVersionDescriptionProvider>();
        
        foreach (var apiVersion in provider.ApiVersionDescriptions)
        {
            ConfigureVersionedDescription(options, apiVersion);
        }

        var xmlCommentsPath = Assembly.GetExecutingAssembly()
            .Location.Replace("dll", "xml");
        options.IncludeXmlComments(xmlCommentsPath);

        options.OperationFilter<SwaggerOperationFilter>();
        options.DocumentFilter<SwaggerDocumentFilter>();
    });
}
        
private void ConfigureVersionedDescription(SwaggerGenOptions options, ApiVersionDescription apiVersion)
{
    //In production code you should probably use a seperate class to get these version descriptions
    var dictionairy = new Dictionary<string, string> 
    {
        { "1.0", "This API features several endpoints showing different API features for API version V1" },
        { "2.0", "This API features several endpoints showing different API features for API version V2" }
    };

    var apiVersionName = apiVersion.ApiVersion.ToString();
    options.SwaggerDoc(apiVersion.GroupName,
        new Info()
        {
            Title = "NetCore.WebApiExample",
            Version = apiVersionName,
            Description = dictionairy[apiVersionName] 
        });
}
```

Finally change the Configure method as shown below to enable the version selection in Swagger UI.(Notice the IApiVersionDescriptionProvider provider parameter is added to the method signature.)

```c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env, IApiVersionDescriptionProvider provider)
{
    app.UseSwagger();

    app.UseSwaggerUI(c =>
    {
        foreach (var apiVersion in provider.ApiVersionDescriptions
                 .OrderBy(version => version.ToString()))
        {
            c.SwaggerEndpoint(
                $"/swagger/{apiVersion.GroupName}/swagger.json",
                    $"NetCore.WebApiExample {apiVersion.GroupName}"
            );
        }
    });

    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

After this configuration the Swagger UI will have the different versions available in the dropdown to the left. Also notice that the version 2 will only contain the routes and models defined for version 2. 

When versioning is enabled the api-version query parameter input field will be automatically added to all methods in Swagger UI.

Swagger UI for version 1:

![[Swagger UI API Version 1]]({{"/assets/images/20190119/SwaggerUIWebAPIV1.png" | relative_url }})

Swagger UI for version 2:

![[Swagger UI API Version 2]]({{"/assets/images/20190119/SwaggerUIWebAPIV2.png" | relative_url }})

If you use document or operation filters (as explained in [my previous post](https://samanthaneilen.github.io/2018/12/08/Using-and-extending-swagger.json-for-API-documentation.html "my blogpost regarding Swagger/OpenApi/Swashbuckle constraints")) you may get some unintended behavior when they are executed based on strings and names that overlap over versions.  

For example look at the document filter below:

```c#
using Swashbuckle.AspNetCore.Swagger;
using Swashbuckle.AspNetCore.SwaggerGen;
using System;
using System.Collections.Generic;
using System.Linq;

namespace NetCore.WebApiExample.MiddleWare.Swagger
{
    public class SwaggerDocumentFilter : IDocumentFilter
    {
        private readonly List<Tag> _tags = new List<Tag>
        {
            new Tag { 
            	Name = "RoutingApi", 
            	Description = "This is a description for the api routes" 
            }
        };

        public void Apply(SwaggerDocument swaggerDoc, DocumentFilterContext context)
        {
            if (swaggerDoc == null)
            {
                throw new ArgumentNullException(nameof(swaggerDoc));
            }

            swaggerDoc.Tags = GetFilteredTagDefinitions(context);
            swaggerDoc.Paths = GetSortedPaths(swaggerDoc);
        }

        private List<Tag> GetFilteredTagDefinitions(DocumentFilterContext context)
        {
            //Filtering ensures route for tag is present
            var currentGroupNames = context.ApiDescriptions
            	.Select(description => description.GroupName);
            return _tags.Where(tag => currentGroupNames.Contains(tag.Name)).ToList();
        }

        private IDictionary<string, PathItem> GetSortedPaths(
        	SwaggerDocument swaggerDoc)
        {
            return swaggerDoc.Paths.OrderBy(pair => pair.Key)
            	.ToDictionary(pair => pair.Key, pair => pair.Value);
        }
    }
}
```

The Tag list contains a hardcoded name here that is not present when switching to version 2 of the API (see the screenshots of Swagger UI earlier in the post). If you do not use the context.ApiDescriptions to check there are routes for the group it will end up adding an empty group for version 2. If I did not add the filtering the Swagger UI page for version 2 would have looked as shown in the screenshot below.

![[Swagger UI with empty Tag]]({{"/assets/images/20190119/SwaggerUIEmptyTag.png" | relative_url }})

### Resources 

For more information on all the features and configuration of the versioning packages visit the link below.

 [Microsoft.Asp.Net.WebApi.Versioning github repository wiki](https://github.com/Microsoft/aspnet-api-versioning/wiki "Microsoft.Asp.Net.WebApi.Versioning github repository wiki") 

 