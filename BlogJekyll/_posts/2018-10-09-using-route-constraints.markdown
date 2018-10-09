---
layout: post
title:  "Using route constraints for input validation and improved security"
date:   2018-10-09 00:00:00 +0100
tags: MVC WebApi
---
In .Net web applications you use the routing system to expose URLs and endpoints in your application. If your method has a framework value type like int then the routing engine will automatically parse your inputs to these types.  

Also if you the default convention based route in a .Net Framework MVC web application uses the convention below:

{% highlight C# %}
routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
);
{% endhighlight %}

This means that if I have the controller with a method as shown below:

{% highlight C# %}
public class MyController : Controller
{
    public string Index(int id)
    {
        return id.ToString();
    }
}
{% endhighlight %}
If I pas a numeric value that does not map to an int32 (for example max Int32 +1 (= 2147483648)) so <span class="link-style">http://localhost/My/Index/2147483648</span> or if I use <span class="link-style">http://localhost/My/Index/abc</span>, I get a server error saying that I passed a null value for a non-nullable integer. This error occurs because the input value could not be parsed to the correct value type.
 
After your method gets hit with an valid int32 value, you may also want to do some input validation. For example: is my integer input a positive number. When using string inputs you may want to validate and execute different code depending on what input was given. An example of this is shown below:

{% highlight C# %}
public IActionResult Get(int input)
{
    if (input < 0)
    {
        throw new ArgumentException(nameof(input), "Input has to be a valid positive integer");
    }

    return businessLogic.ProcessInput(input);
}
{% endhighlight %}

{% highlight C# %}
public IActionResult Get(string input)
{
    if (string.IsNullOrEmpty(input))
    {
        throw new ArgumentNullException(nameof(input));
    }

    if(input.Contains("@"))
    {
        //return something
    }
    else
    {
        // return something else
    }
}
{% endhighlight %}

By using route constraints you can avoid cluttering controllers with validation code and routing logic inside your controller methods. Using route constraints will also add security benefits. If a route is not found the processing will stop quite early in the processing pipeline and return a 404 not found exception. This 404 exception will also never include sensitive internal information about a specific method that was being processed.

In this blogpost I will be describing the default route constraints provided by the .Net Frameworks and how to create custom constraints to make sure a method or route only gets hit by the intended input.

I have created a [reference project on GitHub](https://github.com/SamanthaNeilen/WebApiExamples "reference project on GitHub"){:target="_blank"} where all code snippets described in the rest of this blogpost can be found in a running project including integration tests demonstrating the working and denied routes.

### Route constraints in a .Net Framework web application for MVC controllers
Create a new .Net Framework web application using the MVC template. Next navigate to the Controllers/Home.cs file. The parameterless methods for showing the homepage, contact and about page are already listed. The default template uses the convention based routing by default. The default route template is defined in App_Start/RouteConfig.cs.

To add another route using a route constraint to the convention based routes, add the code as shown below:

RouteConfig.cs (make sure to add this above the default route)

{% highlight C# %}
routes.MapRoute(
    name: "GetNumber",
    url: "GetNumber/{input}",
    defaults: new { controller = "Home", action = "GetNumber" },
    constraints: new { input = @"\d{1,3}" }
);
{% endhighlight %}
 
HomeController.cs

{% highlight C# %}
public string GetNumber(int input)
{
    return $"Route is HomeController.GetNumber(({nameof(input)})";
}
{% endhighlight %}


With this defined route you will expose the <span class="link-style">http://localhost/GetNumber/{number}</span> route where the number parameter is a number of 1 to 3 digits. The constraints parameter in the MapRoute method accepts regular expressions and will route any call to the GetNumber method if the digits match. If the route does not match (for example {number} is 4 digits) no route will be found and you will get a HTTP 404 not found error when calling the GetNumber url.

Next if your logic is too complex for a regular expression you can choose to define a custom RouteConstraint as shown in the code snippets below:

RouteConfig: (again make sure to add this above the default route)
{% highlight C# %}
routes.MapRoute(
    name: "GetEmail",
    url: "GetEmail/{email}",
    defaults: new { controller = "Home", action = "GetEmail" },
    constraints: new { Email = new EmailRouteContraint() }
);
{% endhighlight %}

HomeController.cs:
{% highlight C# %}
public string GetEmail(string email)
{
    return $"Route is HomeController.GetEmail(({nameof(email)})";
}
{% endhighlight %}

EmailRouteContraint.cs (this is a new class file added to the project)

{% highlight C# %}
using System;
using System.Globalization;
using System.Web;
using System.Web.Routing;
using Utilities;

namespace Framework.WebApiExample.Custom
{
    public class EmailRouteContraint : IRouteConstraint
    {
        public bool Match(HttpContextBase httpContext, Route route, string parameterName, 
                RouteValueDictionary values, RouteDirection routeDirection)
        {
            RouteParameterInputValidation(httpContext, route, parameterName, values);

            if (values.TryGetValue(parameterName, out var routeValue))
            {
                var parameterValueString = Convert.ToString(routeValue, CultureInfo.InvariantCulture);
                return parameterValueString.IsEmail();
            }

            return false;
        }

        private void RouteParameterInputValidation(HttpContextBase httpContext, Route route, 
                string parameterName, RouteValueDictionary values)
        {
            //validate input params  
            if (httpContext == null)
            {
                throw new ArgumentNullException(nameof(httpContext));
            }

            if (route == null)
            {
                throw new ArgumentNullException(nameof(route));
            }

            if (parameterName == null)
            {
                throw new ArgumentNullException(nameof(parameterName));
            }

            if (values == null)
            {
                throw new ArgumentNullException(nameof(values));
            }
        }
    }
}
{% endhighlight %}

IsEmail extension method implementation:

{% highlight C# %}
namespace Utilities
{
    public static class Extensions
    {
        public static bool IsEmail(this string value)
        {
            return value.Contains("@");
        }
    }
}
{% endhighlight %}

Now we’ve set up a route for <span class="link-style">http://localhost/GetEmail/{email}</span> where the input string for email will only match if the value has a @ symbol in it. So just <span class="link-style">http://localhost/GetEmail/abc</span> will return a 404 not found result, but <span class="link-style">http://localhost/GetEmail/abc@</span> will route to our GetEmail method in the HomeController.cs file. 

Convention based routing has some limitations for route constraints and also the route templates may unintentionally overlap and result in unexpected behavior. (If we did not specify the GetEmail of GetNumber paths in the url, the default route would have used our input and try to route it to one of our methods). Also attribute routing already has default constraint options for basic value type constraints. 

For more information on convention based and attribute based routing and conventions for defining a route see the Microsoft docs pages about routing. 

- [Microsoft docs page for convention based routing](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/controllers-and-routing/asp-net-mvc-routing-overview-cs "Microsoft docs page for convention based routing"){:target="_blank"}.
- [Microsoft docs page for attribute based routing](https://docs.microsoft.com/en-us/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#adding-route-attributes "Microsoft docs page for attribute based routing"){:target="_blank"}.

To enable attribute based routing, add the snippet below to the RouteConfig file. Next add a number method using a Route attribute with a default inline constraint as shown below.

RouteConfig: 
{% highlight C# %}
routes.MapMvcAttributeRoutes();
{% endhighlight %}

HomeController.cs:
{% highlight C# %}
[Route("Attr/{number:range(0,2147483647)}")]
public string GetNumberAttributeRoute(int number)
{
    return $"Route is HomeController.Get({nameof(number)})";
}
{% endhighlight %}

Now we have a <span class="link-style">http://localhost/Attr/{number}</span> route defined. The default route constraint will ensure that only a valid positive integer will be accepted. Negative input will result in a 404 not found result. Also if you did not specify a constraint, an input larger than Int32 would have routed to this method and the input would have been 0 as the number would not have been parsable to a valid int32.

For a table of the default inline constraints for attribute routing see: [the Microsoft docs page for default inline constraints](https://docs.microsoft.com/en-us/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#route-constraints "the Microsoft docs page for default inline constraints"){:target="_blank"}. These default constraints do not seem to work in the convention based routing constraints parameter. When using convention based routing you will only be able to use regex or custom route constraint implementations.

To add a custom constraint to an attribute routing template, change the call to the MapMvcAttributeRoutes method to the snippet shown below and add a method using the custom constraint in the HomeController.

RouteConfig: 
{% highlight C# %}
var constraintsResolver = new DefaultInlineConstraintResolver();
constraintsResolver.ConstraintMap.Add("Email", typeof(EmailRouteContraint));
routes.MapMvcAttributeRoutes(constraintsResolver);
{% endhighlight %}

HomeController.cs:
{% highlight C# %}
[Route("Attr/{email:Email}")]
public string GetEmailAttributeRoute(string email)
{
    return $"Route is HomeController.Get({nameof(email)})";
}
{% endhighlight %}

Here we map the string “Email” to our custom constraint implementation. Now we can use that string in our Route template as “{parametername:Email}”. When using multiple custom constraints you have to map them to unique names to use in your routes. The implementation above maps the url <span class="link-style">http://localhost/Attr/{email}</span> to our method. So now we can use <span class="link-style">http://localhost/Attr/abc@</span> as a valid route but the constaint will ensure that <span class="link-style">http://localhost/Attr/abc</span> will still return a 404 not found.

###  Route constraints in a .Net Framework web application for WebApi controllers
In the .Net Framework projects, MVC Controllers and WebApi Controllers work with different implementations. When adding a new controller and selecting a WebApi controller template, a WebApiConfig.cs file will appear in the App_Start folder. As you compare that file with the RouteConfig.cs you will see that the methods use different types with different methods to configure routing. 
(When adding a WebApi controller to an MVC project you also need add the GlobalConfiguration.Configure(WebApiConfig.Register); line to your global.asax file to enable the WebApi routing)

We can add a WebApi controller to the project and define some routes showing the attribute route constraints with almost the same code as we used for the MVC controller. 

Global.asax: (full Global.asax is shown but the only change I made was the call to the WebApiConfig.Register method)
{% highlight C# %}
using System.Web.Http;
using System.Web.Mvc;
using System.Web.Optimization;
using System.Web.Routing;

namespace Framework.WebApiExample
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            GlobalConfiguration.Configure(WebApiConfig.Register);
            FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            BundleConfig.RegisterBundles(BundleTable.Bundles);
        }
    }
}
{% endhighlight %}

WebApiConfig.cs: 
{% highlight C# %}
using Framework.WebApiExample.Custom;
using System.Web.Http;
using System.Web.Http.Routing;

namespace Framework.WebApiExample
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            var constraintsResolver = new DefaultInlineConstraintResolver();
            constraintsResolver.ConstraintMap.Add("Email", typeof(EmailHttpRouteConstraint));
            config.MapHttpAttributeRoutes(constraintsResolver);

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
    }
}
{% endhighlight %}

EmailHttpRouteConstraint.cs:
{% highlight C# %}
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Net.Http;
using System.Web.Http.Routing;
using Utilities;

namespace Framework.WebApiExample.Custom
{
    public class EmailHttpRouteConstraint : IHttpRouteConstraint
    {
        public bool Match(HttpRequestMessage request, IHttpRoute route, string parameterName, 
                IDictionary<string, object> values, HttpRouteDirection routeDirection)
        {
            RouteParameterInputValidation(request, route, parameterName, values);

            if (values.TryGetValue(parameterName, out var routeValue))
            {
                var parameterValueString = Convert.ToString(routeValue, CultureInfo.InvariantCulture);
                return parameterValueString.IsEmail();
            }

            return false;
        }

        private void RouteParameterInputValidation(HttpRequestMessage request, IHttpRoute route, 
                string parameterName, IDictionary<string, object> values)
        {
            //validate input params  
            if (request == null)
            {
                throw new ArgumentNullException(nameof(request));
            }

            if (route == null)
            {
                throw new ArgumentNullException(nameof(route));
            }

            if (parameterName == null)
            {
                throw new ArgumentNullException(nameof(parameterName));
            }

            if (values == null)
            {
                throw new ArgumentNullException(nameof(values));
            }
        }

    }
}
{% endhighlight %}

RoutingApiController.cs:
{% highlight C# %}
using System.Web.Http;

namespace Framework.WebApiExample.Controllers
{
    [RoutePrefix("api/routingapi")]
    public class RoutingApiController : ApiController
    {
        [HttpGet]
        [Route("{validPositiveInt32Value:range(0,2147483647)}")]
        public IHttpActionResult Get(int validPositiveInt32Value)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(validPositiveInt32Value)}");
        }

        [HttpGet]
        [Route("{valueMatchingRegex:regex(^(.+)(_)(.+)$)}")]
        public IHttpActionResult GetByRegex(string valueMatchingRegex)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(valueMatchingRegex)})");
        }

        [HttpGet]
        [Route("{email:Email}")]        
        public IHttpActionResult Get(string email)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(email)})");
        }
    }
}
{% endhighlight %}

As you may have noticed in the code above. The email custom route constraint for a WebApi controller implements the IHttpRouteConstraint interface instead of the IRouteConstraint interface. Be aware when intermingling MVC and WebApi controllers in a .Net Framework web application project that other attributes (like authorization) will also usually require different implementations to be used for a MVC and a WebApi controller.

By adding the RoutingApiController listed above the following routes become availlable:

- <span class="link-style">http://localhost/api/routingapi/{ValidPositiveInt32}</span>
- <span class="link-style">http://localhost/api/routingapi/{StringSplitByUnderscore}</span>
- <span class="link-style">http://localhost/api/routingapi/{StringContaining@Symbol}</span>


###  Route constraints in a .Net Core web application

In the .Net Core pipeline MVC and WebApi controllers now share the same base classes. The setup for routing is also now centralized in the Startup classes. The code snippets below how to set up the routing and then show an MVC and a WebApi controller with the same routing constraint concepts that I have explained in the previous sections of this post. 
The code snippets were added to a default .Net Core web application using the Model View Controller project template. The code in the Startup.cs is needed when adding a custom route constraint in the template. If you only use the route constraints that can be resolved by the DefaultInlineConstraintResolver, the code for setting up the RouteOptions configuration can be omitted.

Startup.cs 
{% highlight C# %}
using Microsoft.AspNetCore.Routing;
using NetCore.WebApiExample.MiddleWare;

...

public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        // This lambda determines whether user consent for non-essential cookies is needed for a given request.
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.None;
    });

    services.Configure<RouteOptions>(routeOptions =>
    {
        routeOptions.ConstraintMap.Add("Email", typeof(EmailRouteContraint));
    });

    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
{% endhighlight %}

EmailRouteConstraint.cs 
{% highlight C# %}
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Routing;
using System;
using System.Globalization;
using Utilities;

namespace NetCore.WebApiExample.MiddleWare
{
    public class EmailRouteContraint : IRouteConstraint
    {
        public bool Match(HttpContext httpContext, IRouter route, string routeKey, RouteValueDictionary values, RouteDirection routeDirection)
        {
            RouteParameterInputValidation(httpContext, route, routeKey, values);

            if (values.TryGetValue(routeKey, out var routeValue))
            {
                var parameterValueString = Convert.ToString(routeValue, CultureInfo.InvariantCulture);
                return parameterValueString.IsEmail();
            }

            return false;
        }

        private void RouteParameterInputValidation(HttpContext httpContext, IRouter route, string routeKey, RouteValueDictionary values)
        {
            //validate input params  
            if (httpContext == null)
            {
                throw new ArgumentNullException(nameof(httpContext));
            }

            if (route == null)
            {
                throw new ArgumentNullException(nameof(route));
            }

            if (routeKey == null)
            {
                throw new ArgumentNullException(nameof(routeKey));
            }

            if (values == null)
            {
                throw new ArgumentNullException(nameof(values));
            }
        }
    }
}
{% endhighlight %}

HomeController.cs (.Net Core MVC Controller)
{% highlight C# %}
using Microsoft.AspNetCore.Mvc;

namespace NetCore.WebApiExample.Controllers
{
    public class HomeController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }

        [HttpGet]
        [Route("{email:Email}")]
        public IActionResult Get(string email)
        {
            return Ok($"Route is HomeController.Get({nameof(email)})");
        }
    }
}
{% endhighlight %}

RoutingApiController.cs (.Net Core WebApi Controller)
{% highlight C# %}
using Microsoft.AspNetCore.Mvc;

namespace NetCore.WebApiExample.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RoutingApiController : ControllerBase
    {
        [HttpGet]
        [Route("{email:Email}")]
        public IActionResult Get(string email)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(email)})");
        }

        [HttpGet]
        [Route("{inputMatchingRegex:regex(^(.+)(_)(.+)$)}")]
        public IActionResult GetByRegex(string inputMatchingtRegexForValueContainingUnderscore)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(inputMatchingtRegexForValueContainingUnderscore)})");
        }

        [HttpGet]
        [Route("{validPositiveInt32Value:range(0,2147483647)}")]
        public IActionResult Get(int validPositiveInt32Value)
        {
            return Ok($"Route is RoutingApiController.Get({nameof(validPositiveInt32Value)}");
        }
    }
}
{% endhighlight %}

### Testing routing behavior with integration tests
To ensure your routes work as intended and faulty input does not lead to errors you can write a set of integration tests. I used a .Net Core test project combined with the NUnit testing framework to write the tests shown below.

TestClass: (the RouteDriver implementation is listed below the TestClass code snippet )
{% highlight C# %}
using IntegrationTests.TestDriver;
using NUnit.Framework;

namespace IntegrationTests
{
    /// <summary>
    /// Be sure to have local instance of the Framework.WebApiExample website running on the url provided in the _localHostUrl constant.
    /// </summary>
    [TestFixture]    
    public class FrameworkMvcRouteTests
    {
        private const string _localHostUrl = "http://localhost:55528/";
        private RouteTestDriver _testDriver;

        [SetUp]
        public void Setup()
        {
            _testDriver = new RouteTestDriver(_localHostUrl);
        }

        [TearDown]
        public void TearDown()
        {
            _testDriver = null;
        }

        [Test]
        [TestCase("")] //Home/Index route    
        [TestCase("GetEmail/abc@")]
        [TestCase("GetEmail/abc@abc")]
        [TestCase("GetEmail/@abc")]
        [TestCase("GetNumber/0")]
        [TestCase("GetNumber/12")]
        [TestCase("GetNumber/123")]
        public void FrameworkMvc_Given_Relative_MVC_Convention_Based_Route_Should_Be_Valid_Route(string relativeUrl)
        {
           Assert.IsTrue(_testDriver.UrlReturnsSuccessStatusCode(relativeUrl));                                          
        }

        [Test]        
        [TestCase("abc")]
        [TestCase("GetEmail")]
        [TestCase("GetEmail/abc")]
        [TestCase("GetEmail/123")]
        [TestCase("GetNumber")]
        [TestCase("GetNumber/1234")]
        public void FrameworkMvc_Given_Relative_MVC_Convention_Based_Route_Should_Return_404_Not_Found_StatusCode(string relativeUrl)
        {
            Assert.IsTrue(_testDriver.UrlReturns404NotFoundStatusCode(relativeUrl));
        }


        [Test]   
        [TestCase("Attr/abc@")]
        [TestCase("Attr/abc@abc")]
        [TestCase("Attr/@abc")]
        [TestCase("Attr/0")]
        [TestCase("Attr/12345")]
        [TestCase("Attr/2147483647")]
        public void FrameworkMvc_Given_Relative_MVC_Attribute_Based_Route_Should_Be_Valid_Route(string relativeUrl)
        {
            Assert.IsTrue(_testDriver.UrlReturnsSuccessStatusCode(relativeUrl));
        }

        [Test]
        [TestCase("Attr")]
        [TestCase("Attr/abc")]
        [TestCase("Attr/-1")]
        [TestCase("Attr/2147483648")]
        public void FrameworkMvc_Given_Relative_MVC_Attribute_Based_Route_Should_Return_404_Not_Found_StatusCode(string relativeUrl)
        {
            Assert.IsTrue(_testDriver.UrlReturns404NotFoundStatusCode(relativeUrl));
        }
    }
}
{% endhighlight %}

RouteTestDriver.cs (this class is a helper to centralize code used for multiple tests)
{% highlight C# %} 
using System;
using System.Net;
using System.Net.Http;

namespace IntegrationTests.TestDriver
{
    public class RouteTestDriver
    {
        private string _localHostBaseUrl;

        public RouteTestDriver(string localHostBaseUrl)
        {
            if (string.IsNullOrWhiteSpace(localHostBaseUrl))
            {
                throw new ArgumentException(nameof(localHostBaseUrl));
            }

            _localHostBaseUrl = AppendBackSlashIfNeeded(localHostBaseUrl);

        }

        public bool UrlReturnsSuccessStatusCode(string relativeUrlForTest)
        {
            var httpClient = HttpClientFactory.GetInstance();
            var uriToTest = new Uri(_localHostBaseUrl + (relativeUrlForTest ?? string.Empty));
            var httpCallResult = httpClient.GetAsync(uriToTest).Result;
            return httpCallResult.IsSuccessStatusCode;

        }

        public bool UrlReturns404NotFoundStatusCode(string relativeUrlForTest)
        {

            var httpClient = HttpClientFactory.GetInstance();
            var uriToTest = new Uri(_localHostBaseUrl + (relativeUrlForTest ?? string.Empty));
            var httpCallResult = httpClient.GetAsync(uriToTest).Result;
            return httpCallResult.StatusCode == HttpStatusCode.NotFound;

        }

        private string AppendBackSlashIfNeeded(string localHostBaseUrl)
        {
            return !localHostBaseUrl.EndsWith("/")
                ? localHostBaseUrl + "/"
                : localHostBaseUrl;
        }       
    }

    internal static class HttpClientFactory
    {
        private static HttpClient _instance;

        public static HttpClient GetInstance()
        {
            if (_instance == null)
            {
                _instance = new HttpClient();
            }

            return _instance;
        }
    }
}
{% endhighlight %}

These tests can help you make sure that a route either works (by returning a success status code) or is not found (by returning a 404 not found status). I made sure to explicitly check for 404 not found statuses for the “failure” cases and not just check if the success status code was false. This ensures no unwanted internal server errors are returned. Internal server errors are unwanted behavior and if error handling is set up in the wrong way, they may even expose internals of your application that you do not want your users to see.

### Microsoft Docs reference links:

- [Microsoft docs page for convention based routing](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/controllers-and-routing/asp-net-mvc-routing-overview-cs "Microsoft docs page for convention based routing"){:target="_blank"}.
- [Microsoft docs page for creating a custom route constraint for convention based routing](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/controllers-and-routing/creating-a-route-constraint-cs "Microsoft docs page for creating a custom route constraint for convention based routing"){:target="_blank"}.
- [Microsoft docs page for attribute based routing](https://docs.microsoft.com/en-us/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2#adding-route-attributes "Microsoft docs page for attribute based routing"){:target="_blank"}.
- [Microsoft docs page for routing in .Net Core 2.1](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing?view=aspnetcore-2.1 "Microsoft docs page for routing in .Net Core 2.1"){:target="_blank"}.

