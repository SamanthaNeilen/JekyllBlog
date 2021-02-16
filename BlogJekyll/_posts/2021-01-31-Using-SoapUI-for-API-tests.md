---
layout: post
title:  "Using SoapUI for API tests"
date:   2021-01-31 00:00:00 +0100

tags: WebAPI
---

SoapUI is an API testing tool like Postman. It’s an open-source and free API testing tool. It can be used for both SOAP and REST endpoints. You can pretty easily make repeatable calls, as you can with Postman, and save the files in source control with your API.  

The features that make it stand out for me is the fact that you can make a TestSuite that calls several tests and the local MockServices. 

The TestSuites make use of the out-of-the-box assertions on the responses and these assertions can be extended by either switching to a paid version or by using Groovy scripts. You can define the URL for TestSuites in project parameters so you can easily point the TestSuite to a different URL or environment.

You can define a local MockService to run on your local machine and have it return payloads to the application that you are testing or running. This can be very helpful when you are creating an application for an API that doesn't exist yet.

The drawbacks of SoapUI are the steeper learning curve for getting started compared to Postman. The fact that these capabilities are very useful for local development but not for running the tests automatically in a deployment pipeline or deploying a mock endpoint to a test environment.

For more robust repeatable testing in a deployment pipeline, writing test code using C# combined with one of the testing frameworks and Selenium and/or Specflow will give you the most options. However Postman, and for extended capabilities, SoapUI will add a lot of benefits in regards to local and regression testing.

**Table of contents:**

* Table of Contents
{:toc}
<br/><br/>

### Creating and sharing a project 

When opening SoapUI, use the Empty button to create a new project. You can right-click the new project in the navigator pane to rename the project and to add a new Rest service from URI (or even a SOAP endpoint with a WSDL) . After creating the project you can add the REST service as shown in the screenshot below.

![[New project]]({{"/assets/images/20210131/NewProject.png" | relative_url }})<br/><br/>

![[New REST-service]]({{"/assets/images/20210131/NewRestService.png" | relative_url }})<br/><br/>

This will create a service with an endpoint in de project to which you can add other endpoints and several different requests per endpoint. 

Double-click a specific request to open the Request editor to execute the request and view the response. Use the play button to execute the request and view the results on the right side of the window. You may have to select the JSON or HTML tab to see the response.  

![[New service endpoint request]]({{"/assets/images/20210131/NewServiceEndpointRequest.png" | relative_url }})<br/><br/>

Add extra resources (for example /API/other-path) by right-clicking the service and select the option New Resource.

Add extra methods to call the resource for different parameters and HTTP verbs by right-clicking the resource and selecting New Method (or New Child to create a sub-resource and then new Method).

Use parentheses like {id} to create a parameter in the path.

![[Resource with template parameter]]({{"/assets/images/20210131/ResourceWithTemplateParameter.png" | relative_url }})<br/><br/>

Add extra requests to a method for different parameter values by right-clicking the method and then choosing the New Request option.

For requests using a JSON body be sure to add a Content-Type: application/JSON header to the request to ensure proper interpretation of the body content. To ensure you receive a JSON response when multiple response types are possible add an Accept: application/JSON header to the request. The Request editor contains a header tab to add these headers. See the screenshot below.

![[Add header]]({{"/assets/images/20210131/AddHeader.png" | relative_url }})<br/><br/>

In the screenshot below, I have added a request for each of the Values controller endpoints to the project in the Navigator pane. The screenshot also shows a POST request and how all the parameters and settings are set in the Request editor.

![[Project with all requests and Requesteditor parameters]]({{"/assets/images/20210131/ProjectWithAllRequestAndRequestEditorParameters.png" | relative_url }})<br/><br/>

When you have to wish to save the project for sharing right-click the project file in the navigator pane and select the Save Project option. This will save an XML file containing all the data. Be aware that if you set secrets in the requests that these will be embedded into the project file. 

The project file can be opened using the Import button in the top menu of SoapUI and saved to source control or any collaboration workspace. 

I have marked both options with a red square in the screenshot below: 

![[save and import project]]({{"/assets/images/20210131/SaveAndImportProject.png" | relative_url }})<br/><br/>

I have added my SoapUI project file to [my WebApi examples GitHub repository](https://github.com/SamanthaNeilen/WebApiExamples/tree/soapui-files/Test%20files)  and you can try importing and running the collection on a localhost instance of that project.

### Using custom properties 

SoapUI allows the use of parameters on several levels to easily switch out the parameter values for the entire project, or just on a request or TestSuite.

The parameters or properties can be added to several levels. To see where you can add properties open the properties pane (if it’s not open already) from the left-hand lower side of SoapUI.

![[Properties window]]({{"/assets/images/20210131/PropertiesWindow.png" | relative_url }})<br/><br/>

Whenever the + button is shown you can add your properties or parameters. I like to add environment URLs on the project level. You can assign these to all the requests in the project. When you change the environment parameter used all the requests will execute for the new URL. This makes it easy to switch environments. 

For example, add the project Custom Properties as shown in the screenshot below. As you can see I reference the environment_local property in the environment property. The environment property will be used in the request templates for all the requests so only the value has to be changed to the correct environment property that you want to use. This way you always have all the environment URLs in one place without having to look them up.

![[Project properties environments]]({{"/assets/images/20210131/ProjectPropertiesEnvironments.png" | relative_url }})<br/><br/>

To easily assign the environment property to all the requests in the project, double-click the API node to show the Service Viewer. Go to the Service Endpoints tab, change the fixed Endpoint value to the value {#Project#environment}, set the mode to overwrite, and select the Assign button to assign the parameter to all requests in the project. (Double-check with the Request editor to see that the parameter has indeed been assigned correctly).

![[Assign parameter]]({{"/assets/images/20210131/AssignParameter.png" | relative_url }})<br/><br/>

The syntax ${#ParameterLevel#ParameterName} to assign or use a parameter value will work on most input fields within the project.

Parameters and properties are also invaluable for TestSuites and TestCases. I will explain these features in a later paragraph.

I have added my exported parameterized SoapUI file to [my WebApi examples GitHub repository](https://github.com/SamanthaNeilen/WebApiExamples/tree/soapui-files/Test%20files) and you can try importing and running the project on a localhost instance of that project.

### SoapUI logging

SoapUI provides several types of logging. The screenshot below shows some of the more useful logs. 

The Raw tabs for the requests and responses show the actual URL, headers, and response content of the requests. For the request body, view the request tab. As you can see in the Raw tab for the request the URLs have been replaced by the actual values of the parameters for the send request.

The HTTP logging provides a text log of all the full requests and responses (though the timestamps and newline characters can make it a bit difficult to read at times).

![[Logging panes]]({{"/assets/images/20210131/LoggingPanes.png" | relative_url }})<br/><br/>

These panes will help you track down the response error codes and understand specific requests that you send using the Request editor.

### Creating a TestSuite

TestSuites are a great tool to define some simple regression tests for your API. Create a new TestSuite from the project context menu.

![[New TestSuite]]({{"/assets/images/20210131/NewTestSuite.png" | relative_url }})<br/><br/>

You can add TestCases from the context menu on the new TestSuite. Each test case contains a request or a set of requests that have their responses checked for certain criteria. A test case can be a logical grouping of 1 or several requests with the same purpose like checking certain validation rules.

You could define and use repeatable test steps and change certain parameters for the specific case or step. See [the Soap UI Docs on modularizing your tests](https://www.soapui.org/docs/functional-testing/modularizing-your-tests/) for more guidance.

The image below shows a simple TestCase with a simple Assertion.

![[TestSuite]]({{"/assets/images/20210131/TestSuite.png" | relative_url }})<br/><br/>

Possible test step types are shown in the screenshot below.

![[TestSteps]]({{"/assets/images/20210131/TestSteps.png" | relative_url }})<br/><br/>

The groovy steps provide extensibility by code but it does have a learning curve especially if you are not familiar with the Java programming paradigms.

There are already a lot of very usable out-of-the-box assertions like the HTTP code assertions, string/content comparison, JSON path, and JSON regular expressions assertions.

Be aware that the free version of SoapUI has limitations (as seen for all the greyed out options). 

![[Pro assertions]]({{"/assets/images/20210131/ProAssertions.png" | relative_url }})<br/><br/>

You can easily use the SoapUI menus on either the project to run all TestCases, a single TestSuite, or even a single TestStep with a few simple clicks. 

![[Run tests]]({{"/assets/images/20210131/RunTests.png" | relative_url }})<br/><br/>

Even if you just run these regression tests locally and keep the project file with the codebase you will always have a quick tool to verify whether or not everything is working as intended and even testers that don't have a test automation background can maintain and extend these projects. 

SoapUI also has possibilities for [security tests](https://www.soapui.org/docs/security-testing/getting-started/), [load tests](https://www.soapui.org/docs/load-testing/concept/), and [documenting manual test steps](https://www.soapui.org/docs/functional-testing/teststep-reference/manual-teststep/) for an API. Please see the SoapUI docs linked in the previous sentence for more information on those. 

### Creating a Mockservice 

A MockService is a simple local web service that is started at a certain address. You can tell the service on which endpoints to listen for requests and what responses to send back. This is a great tool when developing a consuming application where the endpoints do not yet exist or to test certain behavior (error responses) from those endpoints.

To create a MockService, again use the context menu on the project node in the navigation pane. 

![[New MockService]]({{"/assets/images/20210131/NewMockService.png" | relative_url }})<br/><br/>

The local URL at which the MockService is run can be changed from the settings.

![[MockService settings]]({{"/assets/images/20210131/MockServiceSettings.png" | relative_url }})<br/><br/>

Now you can add an endpoint (mock action) through the context menu on the MockService. Omit all the query parameters from the endpoint path and set the path parameter to a fixed value. There are ways to add scripts to the MockService so it responds more dynamically to path parameters to return a proper response. See [the SoapUI docs on creating dynamic MockServices](https://www.soapui.org/docs/soap-mocking/creating-dynamic-mockservices/) for more information.

![[NewMock action]]({{"/assets/images/20210131/NewMockAction.png" | relative_url }})<br/><br/>

![[NewMock response]]({{"/assets/images/20210131/NewMockResponse.png" | relative_url }})<br/><br/>

You can define the HTTP status code and the response body for the mock endpoint in the editor. The MockService responses allow you to define sequences of responses so (for the first response define a response message and for all subsequent responses define a very different response). Double-click the mock action to find and change the Dispatch options. The default is Sequence, meaning that all the responses are returned in order and the last response is repeated until the service is stopped or started.

![[MockAction sequence]]({{"/assets/images/20210131/MockActionSequence.png" | relative_url }})<br/><br/>

When a MockService is started you can still edit the responses and status codes while it’s running. SoapUI will return the newly changed response without restarting the MockService first.

Be aware that when consuming the MockService from a local browser that you may have to add extra CORS endpoints to ensure the browser's security to make the requests. See [this Medium article on how to handle CORS from a MockService](https://medium.com/@andrelimamail/how-to-deal-with-cors-in-soap-ui-mock-services-or-anyother-f4cc55b3dccd).