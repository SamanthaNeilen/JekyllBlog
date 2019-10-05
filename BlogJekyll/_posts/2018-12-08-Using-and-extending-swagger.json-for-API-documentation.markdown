---
layout: post
title:  "Using and extending Swagger.json (OpenApi) for API documentation"
date:   2018-12-08 00:00:00 +0100
tags: WebAPI
---
The [OpenAPI specification](https://en.wikipedia.org/wiki/OpenAPI_Specification "OpenAPI specification"){:target="_blank"} (previously known as the Swagger specification) is used to describe a web API in a JSON format. An example format is shown below.
 
{% highlight Json %}
{
  "swagger": "2.0",
  "info": {
    "version": "1.0",
    "title": "WebApplication1",
    "description": "This is a description for WebApplication1"
  },
  "paths": {
    "/api/Values": {
      "get": {
        "tags": [ "Values" ],
        "operationId": "Get",
        "consumes": [],
        "produces": [ "text/plain", "application/json", "text/json" ],
        "parameters": [],
        "responses": {
          "200": {
            "description": "Success",
            "schema": {
              "uniqueItems": false,
              "type": "array",
              "items": { "type": "string" }
            }
          }
        }
      },
      "post": {
        "tags": [ "Values" ],
        "operationId": "Post",
        "consumes": [ "application/json-patch+json", "application/json", "text/json", "application/*+json" ],
        "produces": [],
        "parameters": [
          {
            "name": "value",
            "in": "body",
            "required": false,
            "schema": { "type": "string" }
          }
        ],
        "responses": { "200": { "description": "Success" } }
      }
    }
  },
  "definitions": {}
}
{% endhighlight %}
 
The file describes the endpoint, parameters and returned JSON format for a web API. If you plan to develop an API that will be used by other teams or even 3rd parties outside your company then consider setting up Swagger with the Swashbuckle library for your .NET web API project. 
 
The Swashbuckle library will automaticaly generate the Swagger specification file for your API and even generate a front-end page called Swagger UI. This UI will offer a nice visual overview for your API and also allow user to make calls to the API with build in input validation and view results fot the calls.
Below is screenshot of the UI for the Swagger.json as generated for the definition listed above.
 
<br/><img src="{{"/assets/images/20181208/SwaggerUIExample.png" | relative_url }}" alt="Swagger UI Example"/>
 
In this blogpost I will show you how to how to enable Swagger UI for your .NET Framework and Core web projects. Be aware that in .NET Framework only API Controller methods will be listed. MVC Controllers and actions will not be listed. 
Next I will show some customizations to the default generated Swagger.json and Swagger UI. For these customizations I will only list the .NET Core code. The .NET Framework (as well as the .NET Core) implementation can be found in my [WebApiExamples GitHub repository](https://github.com/SamanthaNeilen/WebApiExamples " WebApiExamples GitHub repository "){:target="_blank"}.

**Table of contents:**
* Table of Contents
{:toc}
 
### Configure the generation of a Swagger.json file
 
To get started install the Swashbuckle NuGet package for a .NET Framework project or Swashbuckle.AspNetCore for a .NET Core project.
 
Next set up the pipeline in the Startup files to enable the generation of a Swagger.json file and the Swagger UI frontend based on the default meta data for your API.
 
For .NET Core, go to the Startup.cs file and add the configuration show below. 
 
{% highlight C# %}
using Swashbuckle.AspNetCore.Swagger;
 
namespace NetCore.WebApiExample
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        { 
            //other configuration omitted...
 
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1",
                    new Info()
                    {
                        Title = "NetCore.WebApiExample",
                        Version = "1.0",
                        Description = "This API features several endpoints showing different API features"
                    });
            });
 
            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
        }
 
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            //other configuration omitted...
 
            app.UseSwagger();
 
            app.UseSwaggerUI(c =>
            {
                c.SwaggerEndpoint(
                 $"/swagger/v1/swagger.json",
                 $"NetCore.WebApiExample 1.0");
            });
           
            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
}
{% endhighlight %}
 
After setting up the Swagger configuration run your project and access the path that now has the SwaggerUI (<span class="link-style">https://localhost:port/swagger</span>). 
The Swagger UI will also have a link to the actual JSON file on which the UI was generated (<span class="link-style">/swagger/v1/swagger.json</span>). The v1 part of this path is the value of the first parameter (name) of the SwaggerDoc method called in the StartUp.cs file. 
 
For a .NET Framework project, a file with a lot of commented out code (App_Start\SwaggerConfig.cs) was added when you installed the NuGet package. This file already enables a default generated Swagger.json endpoint and the Swagger UI frontend.
An example of this file without all the default comments is show below.
 
{% highlight C# %}
using System.Web.Http;
using WebActivatorEx;
using Framework.WebApiExample;
using Swashbuckle.Application;
 
[assembly: PreApplicationStartMethod(typeof(SwaggerConfig), "Register")]
 
namespace Framework.WebApiExample
{
    public class SwaggerConfig
    {
        public static void Register()
        {
            var thisAssembly = typeof(SwaggerConfig).Assembly;
 
            GlobalConfiguration.Configuration
                .EnableSwagger(c => { c.SingleApiVersion("v1", "Framework.WebApiExample"); })
                .EnableSwaggerUi(c => { });
        }
    }
} 
{% endhighlight %}
 
With this configuration the Swagger UI is again reached on <span class="link-style">https://localhost:port/swagger</span>). Note that the Swagger.json for .NET Framework projects is in a different location (<span class="link-style">/swagger/docs/v1</span>).
Again, be aware that in .NET Framework only API Controller methods will be listed. MVC Controllers and actions will not be listed. 
 
### Customizing the Swagger.json
 
All the code shown in this paragraph will only show the .NET Core implementation. The .NET Framework code and configuration will be slightly different. The .NET Framework (as well as the .NET Core) implementation can be found in my [WebApiExamples GitHub repository](https://github.com/SamanthaNeilen/WebApiExamples " WebApiExamples GitHub repository "){:target="_blank"}.
 
The default generated Swagger.json uses the metadata for your classes and methods to generate the specification file. Information such as authentication or other custom headers are not known in the Swagger UI. 
 
You can modify the parameters listed for yourr operation with an extension called an OperationFilter. 
 
An example of an OperationFilter to add a custom header is listed below. You could use for example use this technique to add the Authorization header input fields for secured API.
OperationFilters are run for every discovered method. You can examine the current OperationContext and add conditional logic such as the IsMethodWithHttpGetAttribute function as shown in the example below.
 
{% highlight C# %}
using Microsoft.AspNetCore.Mvc;
using Swashbuckle.AspNetCore.Swagger;
using Swashbuckle.AspNetCore.SwaggerGen;
using System;
using System.Collections.Generic;
using System.Linq;
 
namespace NetCore.WebApiExample.MiddleWare.Swagger
{
    public class SwaggerOperationFilter : IOperationFilter
    {
        public void Apply(Operation operation, OperationFilterContext context)
        {
            ValidateInput(operation, context);
            if (IsMethodWithHttpGetAttribute(context))
            {
                operation.Parameters.Add(new NonBodyParameter
                {
                    Name = "extra",
                    In = "query",
                    Description = "This is an extra querystring parameter",
                    Required = false,
                    Type = "string"
                });
 
                operation.Parameters.Add(new NonBodyParameter
                {
                    Name = "Authorization",
                    In = "header",
                    Description = "Used for certain authorization policies such as Bearer token authentication",
                    Required = false,
                    Type = "string"
                });
            }
        }
 
        private void ValidateInput(Operation operation, OperationFilterContext context)
        {
            if (operation == null)
            {
                throw new ArgumentNullException(nameof(operation));
            }
 
            if (context == null)
            {
                throw new ArgumentNullException(nameof(context));
            }
 
            if (operation.Parameters == null)
            {
                operation.Parameters = new List<IParameter>();
            }
        }
 
        private bool IsMethodWithHttpGetAttribute(OperationFilterContext context)
        {
            return context.MethodInfo.CustomAttributes.Any(attribute => attribute.AttributeType == typeof(HttpGetAttribute));
        }
    }
}
{% endhighlight %}
 
After defining the OperationFilter you will also have to plug the filter into the Swagger processing pipeline as shown below.
 
{% highlight C# %}
public void ConfigureServices(IServiceCollection services)
{ 
    //other configuration omitted...
 
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1",
            new Info()
            {
                Title = "NetCore.WebApiExample",
                Version = "1.0",
                Description = "This API features several endpoints showing different API features"
            });
    });
 
    c.OperationFilter<SwaggerOperationFilter>();
 
    //other configuration omitted...
}
{% endhighlight %}
 
Swagger UI for a Get method before enabling the OperationFilter:
<br/><img src="{{"/assets/images/20181208/BeforeOperationFilter.png" | relative_url }}" alt="Swagger UI before OperationFilter"/>
 
Swagger UI for the same Get method after enabling the OperationFilter:
<br/><img src="{{"/assets/images/20181208/AfterOperationFilter.png" | relative_url }}" alt="Swagger UI after OperationFilter"/>
 
Also you might want to add or modify certain properties or descriptions for the endpoint. To modify a part of the Swagger.json above the "operation" level, use a DocumentFilter.  
 
An example of a DocumentFilter is to add descriptions to the tags. Operations for a Controller will be grouped within a collapsible pane in Swagger UI and the header of the pane is the tag assigned to that grouping. The DocumentFilter listed below will also sort the operations per tag in alphabetical order instead of the default method ordering as discovered within the API controller.
 
{% highlight C# %}
using Swashbuckle.AspNetCore.Swagger;
using Swashbuckle.AspNetCore.SwaggerGen;
using System;
using System.Collections.Generic;
using System.Linq;
 
namespace NetCore.WebApiExample.MiddleWare.Swagger
{
    public class SwaggerDocumentFilter : IDocumentFilter
    {
        public void Apply(SwaggerDocument swaggerDoc, DocumentFilterContext context)
        {
            if (swaggerDoc == null)
            {
                throw new ArgumentNullException(nameof(swaggerDoc));
            }
 
            swaggerDoc.Tags = new List<Tag> {
                    new Tag{ Name = "RoutingApi", Description = "This is a description for the api routes" }
                };
 
            swaggerDoc.Paths = swaggerDoc.Paths.OrderBy(pair => pair.Key).ToDictionary(pair => pair.Key, pair => pair.Value);
        }
    }
}
{% endhighlight %}
  
After defining the DocumentFilter you will also have to plug the filter into the Swagger processing pipeline as shown below.
 
{% highlight C# %}
    public void ConfigureServices(IServiceCollection services)
    { 
        //other configuration omitted...
 
        services.AddSwaggerGen(c =>
        {
            c.SwaggerDoc("v1",
                new Info()
                {
                    Title = "NetCore.WebApiExample",
                    Version = "1.0",
                    Description = "This API features several endpoints showing different API features"
                });
        });
 
        c.OperationFilter<SwaggerOperationFilter>();
        c.DocumentFilter<SwaggerDocumentFilter>();
 
        //other configuration omitted...
    }
{% endhighlight %}
 
Swagger UI method listings before enabling the DocumentFilter:
<br/><img src="{{"/assets/images/20181208/BeforeDocumentFilter.png" | relative_url }}" alt="Swagger UI before DocumentFilter"/>
 
Swagger UI method listings after enabling the DocumentFilter:
<br/><img src="{{"/assets/images/20181208/AfterDocumentFilter.png" | relative_url }}" alt="Swagger UI after DocumentFilter"/>
 
 
Swagger can use certain attributes to enrich the documentation of your API.
Like specifying a return type. These return types will be listed in definitions part of the Swagger.json and will also show in the Swagger UI.
 
{% highlight C# %}
using Microsoft.AspNetCore.Mvc;
using System;
 
namespace NetCore.WebApiExample.Controllers
{
    public class HomeController : Controller
    {        
        [HttpGet]
        [Route("customResponsetype")]
        [ProducesResponseType(200, Type = typeof(MyResponseType))]
        public IActionResult GetCustomResponseType()
        {
            return Ok(new MyResponseType {
                Title = "SomeTitle",
                Data = "SomeData"
 
            });
        }
 
        public class MyResponseType {
            public string Title { get; set; }
 
            public string Data { get; set; }
 
            public int Number { get; set; }
 
            public DateTime Timestamp { get; set; }
        }
    }
}
{% endhighlight %}
 
Swagger UI method listing without the ProduceResponseType attribute:
<br/><img src="{{"/assets/images/20181208/MethodDescriptionBeforeOutputSpecification.png" | relative_url }}" alt="Swagger UI before output specification attribute"/>
 
Swagger UI method listing with the ProduceResponseType attribute (also note the appearance of the models section):
<br/><img src="{{"/assets/images/20181208/MethodDescriptionAfterOutputSpecification.png" | relative_url }}" alt="Swagger UI after output specification attribute"/>
 
Swashbuckle can also use an XML comments file to enrich the descriptions for your endpoints.
 
{% highlight C# %}
using Microsoft.AspNetCore.Mvc;
using System;
 
namespace NetCore.WebApiExample.Controllers
{
    public class HomeController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }
 
        /// <summary>
        /// This route has a XML comments description
        /// </summary>
        /// <param name="email">A valid email</param>
        /// <returns>Single line of text describing the route for the result</returns>
        [HttpGet]        
        [Route("{email:Email}")]
        public IActionResult Get(string email)
        {
            return Ok($"Route is HomeController.Get({nameof(email)})");
        }
    }
}
{% endhighlight %}
 
Next you can output the XML file with the comments in the project properties. In the project properties, go to the Build tab. Be sure to select "All Configurations" the Configuration drop down on the top of the page. Next check the XML file checkbox. In the project properties you can only specify a hardcoded path, but we can change it in the project file to make it dynamic to the build configuration and target framework to make sure the xml file ends up next to your output dll. When enabling the XML comments checkbox warnings will pop-up in the Error List window for every method not decorated with XML comments on the next build. To ignore these warnings, add the value ";1591" to the Suppress Warnings field also in the Build tab of the project properties.
 
Project properties:
<br/><img src="{{"/assets/images/20181208/ProjectPropertiesXmlOutputFile.png" | relative_url }}" alt="Project properties Build tab for XML comments file"/>
 
To make the XML comments file output location variable. Change the DocumentationFile values in the project file as shown below. 
 
{% highlight XML %}
  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>
 
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
    <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\NetCore.WebApiExample.xml</DocumentationFile>
    <NoWarn>1701;1702;1591</NoWarn>
  </PropertyGroup>
 
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|AnyCPU'">
    <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\NetCore.WebApiExample.xml</DocumentationFile>
    <NoWarn>1701;1702;1591</NoWarn>
  </PropertyGroup>
 
{% endhighlight %}
 
After setting the file location, you set the IncludeXmlComments in the Swagger startup configuration to read the comments file with the correct location for the file.
 
{% highlight C# %}
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1",
        new Info()
        {
            Title = "NetCore.WebApiExample",
            Version = "1.0",
            Description = "This API features several endpoints showing different API features"
        });
 
    var xmlCommentsPath = Assembly.GetExecutingAssembly().Location.Replace("dll","xml");
    c.IncludeXmlComments(xmlCommentsPath);    
});
 
{% endhighlight %}
 
After this setup, when you navigate to the Swagger page you will see the extra descriptions.
 
Swagger UI method listing without the XML comments file loaded:
<br/><img src="{{"/assets/images/20181208/WithoutXmlComments.png" | relative_url }}" alt="Swagger UI before XML comments"/>
 
Swagger UI method listing with the XML comments file loaded:
<br/><img src="{{"/assets/images/20181208/WithXmlComments.png" | relative_url }}" alt="Swagger UI after XML comments"/>
 
 
### Resources and further documentation
 
- [Swashbuckle github project and documentation](https://github.com/domaindrivendev/Swashbuckle "Swashbuckle github project and documentation"){:target="_blank"}
- [Swashbuckle. AspNetCore github project and documentation](https://github.com/domaindrivendev/Swashbuckle.AspNetCore  "Swashbuckle.AspNetCore github project and documentation"){:target="_blank"}
- [Microsoft docs Swagger page for .NET Core](https://docs.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger "Microsoft docs Swagger page for .NET Core "){:target="_blank"}
- [OpenAPI specification on Wikipedia](https://en.wikipedia.org/wiki/OpenAPI_Specification "OpenAPI specification on Wikipedia"){:target="_blank"}
- [Open API specification GitHub repository](https://github.com/OAI/OpenAPI-Specification "Open API specification GitHub repository"){:target="_blank"}
 

