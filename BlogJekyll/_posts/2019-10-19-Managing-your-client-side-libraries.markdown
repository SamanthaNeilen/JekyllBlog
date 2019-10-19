---
layout: post
title:  "Managing your client-side libraries"
date:   2019-10-19 00:00:00 +0100

tags: JavaScript Stylesheets
---

If you start a new .NET Core Razor Pages or MVC project in Visual Studio, it will create a project containing working starter website. However, due to the speed at which 3rd party libraries update, the included version of Bootstrap and jQuery will probably be out of date. There will also be an empty site stylesheet and JavaScript file with their nested minified equivalents. These placeholder files will not update their minified counterparts if you don't configuring a minifyer.  In this blogpost I explain the tags used for referencing client-side libraries, how to use LibMan for 3rd party client-side libraries and how to use BuildBindlerMinifier to bundle and minify files in your project. 


**Table of contents:**
* Table of Contents
{:toc}
### How the script and stylesheets are referenced in the view files.

.NET Core projects use the syntax below in the _Layout.cshtml to include 3rd party client-side libraries.

```html
<!DOCTYPE html>
<html>
<head> 
    <environment include="Development">
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    </environment>
    <environment exclude="Development">
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"
              asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
              asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute"
              crossorigin="anonymous"
              integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T"/>
    </environment>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>

    <environment include="Development">
        <script src="~/lib/jquery/dist/jquery.js"></script>
        <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.js"></script>
    </environment>
    <environment exclude="Development">
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"
                asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
                asp-fallback-test="window.jQuery"
                crossorigin="anonymous"
                integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8=">
        </script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.bundle.min.js"
                asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"
                asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
                crossorigin="anonymous"
                integrity="sha384-xrRywqdh3PHs8keKZN+8zzc5TX0GRTLCcmivcbNJWm2rs5C8PRhcEn3czEjhAO9o">
        </script>
    </environment>
    <script src="~/js/site.js" asp-append-version="true"></script>

    @RenderSection("Scripts", required: false)
</body>
</html>
```

Notice the environment elements. When using the development environment, the local unminified files are used. For all other environments it uses a CDN (content delivery network) link to a minified file. The value of "environment" is an environment variable and can be set via the Debug tab in the property pages or by changing the Properties/launchsettings file. For more information on using the environment variable see [the Microsoft docs page on this variable](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments).

The asp-fallback-test attribute will check if a CSS class or JavaScript function for that specific file is available. If the test fails it will load the (local) script file that is defined in the asp-fallback-src attribute. For more information on the benefits of using the CDN and to dive deeper into the fallback attributes see the [Microsoft Docs page for the Link Tag Helper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/link-tag-helper).

The crossorigin attribute is always anonymous unless a cookie or user credentials are required to access the file from the remote location. For more information on this attribute see the [MDN docs page for the crossorigin attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_settings_attributes). 

The integrity attribute is probably the most important to know about in regard to upgrading library versions. The value of this attribute is a cryptographic hash that matches the contents for the file referenced file. If the referenced file and the hash no longer match up, the browser will see the file as security compromise and block it. For more information on the integrity attribute see [the MDN docs page for the integrity attribute](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity). This page also has instructions and links for getting the appropriate hash for the files that you are trying to reference and embed in your website. 

### Managing 3rd party Client-Side libraries with LibMan

We used to manage front-end libraries with NuGet or include files manually. These days Visual Studio has support for LibMan for managing client-side libraries from several sources in a .NET Core web project. LibMan already has a basic ["Getting started" guide on Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/client-side/libman/libman-vs) so I will not be repeating the information contained there in this post.

If you need packages from NPM that have multiple dependencies, then LibMan (unpkg/jsDelivr source) will not crawl those dependencies and include any  of them. My advice is to just use NPM in that case. Also try to stick to a single client-side package manager when possible for your project. This way the team won't get confused about how the libraries are managed. 

LibMan is good at managing libraries without extra dependencies from cdnjs or other CDNs. If you use the "Add -> Client-Side library" option you get a visual editor however the "Manage Client-Side libraries" option will just open the libman.json file. This is something I found confusing at first. It turns out that the code completion in the JSON file provides the same experience as the visual editor. (It does however take a few minutes for the intellisense to fetch the results from the provider. So, if you're not seeing the list of libraries or files, just wait a minute.)

A few screenshots comparing the visual editor vs the libman.json experience:

![[Comparing library search]]({{"/assets/images/20191019/ComparingLibrarySearch.png" | relative_url }})

![[Comparing version search]]({{"/assets/images/20191019/ComparingVersionSearch.png" | relative_url }})

![[Comparing file selection]]({{"/assets/images/20191019/ComparingFileSelection.png" | relative_url }})

LibMan has out-of-the-box support for several sources but you can use the filesystem source if you need a file from another source. See the screenshot below for an example. The filesystem fallback only works with a complete url to a single file in the library input field. (It will not work with wildcards or a directory).

![[Visual editor filesystem search]]({{"/assets/images/20191019/FilesystemSearch.png" | relative_url }})

### How to bundle and minify your custom scripts 

When you create new .NET Core web project in Visual Studio, an empty site.css and site.js with their minified counterparts will be added. When editing the unminified files, the minified files will not be updated with the changes made. You will need to configure an additional minification tool to enable the minification.

I use the [BuildBundlerMinifier](https://github.com/madskristensen/BundlerMinifier) NuGet package to bundle and minify my own custom script and stylesheet files. To use it, first install the BuildBindlerMinifier package and add a  bundleconfig.json to the project containing the JSON contents as shown below to enable minification of the default files. Note that the input files parameter is an array, so you could provide multiple input files to bundle into one output file.

```json
[
  {
    "outputFileName": "wwwroot/css/site.min.css",
    "inputFiles": [ "wwwroot/css/site.css" ]
  },
  {
    "outputFileName": "wwwroot/js/site.min.js",
    "inputFiles": [ "wwwroot/js/site.js" ],
    "minify": {
      "enabled": true,
      "renameLocals": true
    },
    "sourceMap": false
  }
]
```



If you build the project, the minified files will be updated with the contents from the input files.

For information on how to configure bundling and minification for .NET Core, see [the Microsoft docs pages on bundling and minification](https://docs.microsoft.com/en-us/aspnet/core/client-side/bundling-and-minification).



