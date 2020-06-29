---
layout: post
title:  "Using Postman for quick and repeatable API tests"
date:   2020-06-29 00:00:00 +0100

tags: WebAPI
---

Postman is an API tool that is great for setting up repeatable calls to a REST web service in a short amount of time. When developing an API it’s a good way to check if the API works as expected and saves time in setting up calls every time. Using environment parameters you can also quickly replay the requests on different environments with different parameters. The export functions allow you to store collections and environment files in source control as needed. Using a low-code tool also has the benefit of sharing the calls and files with non-technical testers.

The tool has support for mock API and automated testing but these features require an account and the use of a paid license. When I need those features I usually switch to SoapUI another API testing tool that provides those features for free with no account needed. I will write an article on SoapUI at a later time. SoapUI also provides support for WCF services while Postman focusses on REST calls. 

Due to ease of use and sharing Postman does have my preference over SoapUI until I find I need WCF calls, local mock services, or automated test cases.

More information on [Postman features](https://www.postman.com/) and pricing see the Postman site. You can also download the tool from that website and install it. On the first startup, it will show a dialog prompting to sign in but there is a small skip sign-in link in the pop-up if you do not wish to set up yet another account for a new service.

**Table of contents:**

* Table of Contents
{:toc}
  
<br/><br/>

### Creating and sharing collections

When opening Postman, go to the collections tab and create a new collection. This will create a folder to group requests. It will also allow you to set up certain shared settings like authorization, pre-requests scripts, tests, and variables as can be seen in de screenshot below.

![[Create collection]]({{"/assets/images/20200629/CreateCollection.png" | relative_url }})<br/><br/>

I have added [a simple values controller to my WebApi examples project](https://github.com/SamanthaNeilen/WebApiExamples/blob/api_test_tools/NetCore.WebApiExample/Controllers/ValuesController.cs) as shown below.

``` C#
{%raw%}
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;

namespace NetCore.WebApiExample.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ValuesController : ControllerBase
    {
        [HttpGet]
        public IActionResult Get()
        {
            return Ok(new [] {"value1", "value2"});
        }

        [HttpGet("{id}")]
        public IActionResult Get(int id)
        {
            return Ok($"Requested id: {id}");
        }

        [HttpPost]
        public IActionResult PostObjectParameter([FromBody] KeyValuePair<int,string> keyValuePair)
        {
            return Ok($"Posted values {keyValuePair}");
        }
        
        [HttpPost("{id}")]
        public IActionResult PostValueTypeParameter([FromRoute]int id, [FromBody] KeyValuePair<string,string> value)
        {
            return Ok($"Posted values {{ id: {id}, value: {value} }}");
        }
    }
}
{%endraw%}
```
<br/><br/>

A screenshot listing the requests within postman and the post for the api/values/{id} endpoint is shown below. 

![[Collection requests and POST example]]({{"/assets/images/20200629/CollectionOverview.png" | relative_url }})<br/><br/>

For requests using a JSON body be sure to add a Content-Type: application/json header to the request to ensure proper interpretation of the body content.<br/><br/>

![[Request headers]]({{"/assets/images/20200629/RequestHeaders.png" | relative_url }})<br/><br/>

Postman will add this header automatically when selecting the JSON content-type from the drop-down when using a raw body.<br/><br/>

![[Body content type dropdown]]({{"/assets/images/20200629/BodyContentType.png" | relative_url }}) <br/><br/>

When calling an endpoint protected by authentication/authorization you can either add an Authorization header to the headers tab or use the Authorization tab to fill the header using parameters related to the Authorization method: 

![[Authorization tab dropdown]]({{"/assets/images/20200629/AuthorizationList.png" | relative_url }}) <br/><br/>

When you have to wish to save the collection for sharing click the collection context menu and choose export. This will export a JSON file containing all the data. Be aware that if you set secrets in the requests that these will be embedded into the collection file. 

The collection export file can be imported using the Import function in the top menu of Postman and saved to source control or any collaboration workspace. 

I have marked both options with a red square in the screenshot below: 

![[Import and export collection]]({{"/assets/images/20200629/ImportAndExportCollection.png" | relative_url }}) <br/><br/>

I have added my [exported postman collection file to my WebApi examples GitHub repository](https://github.com/SamanthaNeilen/WebApiExamples/blob/api_test_tools/Test%20files/NetCore%20WebApi%20Examples.postman_collection(no_parameters).json) and you can try importing and running the collection on a localhost instance of that project.

<br/><br/>

### Using environments and parameters

Postman allows the use of parameters and uses environments to easily switch out the parameter value. You can create a file for a local and hosted environment containing the host URL as a parameter.

To do this first create environments using the “Manage environment” option as shown below.

![[Manage environment]]({{"/assets/images/20200629/ManageEnvironment.png" | relative_url }}) <br/><br/>

![[Create environment]]({{"/assets/images/20200629/CreateEnvironment.png" | relative_url }}) <br/><br/>

Next change the host URL in the request to use a parameter placeholder so Postman knows where to replace the variable. Be sure to set the active environment to ensure the parameter is bound to the correct value. The variables can be embedded in the URL or any of the request parameter tabs (authorization, headers, body) as shown in the screenshots below.

![[Parameterized request]]({{"/assets/images/20200629/ParameterizedRequest.png" | relative_url }})<br/><br/>

Environments can also be exported using the download option in the Manage environment pop-up.

![[Download environment]]({{"/assets/images/20200629/DownloadEnvironment.png" | relative_url }})<br/><br/>

The import function in the top left side of Postman can be used for either importing environment or collection files. 

I have added [my exported parameterized postman collection file](https://github.com/SamanthaNeilen/WebApiExamples/blob/api_test_tools/Test%20files/Local%20NetCore%20WebApiExamples.postman_environment.json) and [local environment file](https://github.com/SamanthaNeilen/WebApiExamples/blob/api_test_tools/Test%20files/Local%20NetCore%20WebApiExamples.postman_environment.json) to my WebApi examples GitHub repository and you can try importing and running the collection on a localhost instance of that project.

<br/><br/>

### Using a pre-request script to fetch a bearer token for authorization use cases 

When using OAUTH2 authorization, you can use the pre-requests script on a collection to call a script to fill a variable with a valid token. This will ensure you can always just run the request and not have to worry about retrieving a valid authorization token every time.

[A basic script by Ben Chartrand can be found on Github.](https://gist.github.com/bcnzer/073f0fc0b959928b0ca2b173230c0669)

Be aware that when exporting environment files or postman collections that hard-coded settings and secrets will be exported in the files. In the uploaded files I have created placeholders for the secrets and specific instance settings that correspond with the placeholders in the WebApi configuration files. 

The pre-request script for a collection can be found using the Edit collection context menu setting and then selecting the pre-request script tab.

![[Edit collection option and pre-request script tab]]({{"/assets/images/20200629/EditCollectionPreRequestScript.png" | relative_url }})<br/><br/>

### Postman console

Postman console is a useful feature for troubleshooting and sharing raw requests made by Postman and can be found in the bottom left of the application.

![[Postman console option]]({{"/assets/images/20200629/PostmanConsole.png" | relative_url }}) <br/><br/>

You can inspect the requests made and via the “show raw log” option can change the format to the actual HTTP call that has been sent. The raw format is great for sharing tool independent requests/responses to for example an API vendor for troubleshooting. (Again be aware of any secrets embedded in your requests/responses when sharing this information)

<br/><br/>

### Exporting postman requests as code

Besides sharing the raw requests logged in Postman console you can also share requests by exporting a request as code using the code option below the Save button on a request.

![[Generate code option]]({{"/assets/images/20200629/GenerateCode.png" | relative_url }})<br/><br/>

### Tip for seeing and replaying a browser request

If you have a web application in a browser that calls an API you can usually use the network tab of the developer tools to inspect requests made from the browser to your API. <br/><br/>

![[Browser - network tab - request details]]({{"/assets/images/20200629/BrowserNetworkTabRequestDetails.png" | relative_url }})<br/><br/>

By copying the headers and body for the request to a postman request you can easily replay, edit, and troubleshoot a request without constantly having to fill-out webforms. 