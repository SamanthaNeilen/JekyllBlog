---
layout: post
title:  "Pre-compiled Azure Functions example"
date:   2018-06-08 00:00:00 +0100
tags: Azure AzureFunctions
---
<p>
	Azure Functions are a great way of setting up small asynchronous processing with great parallel processing scaling options and a pay-as-you go pricing model at the cost of some time and memory limitations.
</p> 
<p>There are multiple ways of creating a C# Azure Function: <br/>
- Precompiled Azure Functions are created with a Visual Studio Azure Functions project<br/>
- File based Azure Functions leverage the Azure Function CLI NPM package for development and do not require any specific editor to create.<br/>
</p> 
<p>
	In this blog post I will focus on developing precompiled Azure Functions using Visual Studio. In a next blog post I explain creating an Azure Function using the file based and Azure Functions.
</p>
<p>
	Azure Functions can be developed run and tested locally using the latest version of the Visual Studio 2017 IDE that has the Azure workload installed without the need to upload anything to the cloud. Installing the Azure workload also installs the Azure Storage Emulator to mimic the functionality of a storage account on your local machine.
</p>
<p> 
	I have created an <a href="https://github.com/SamanthaNeilen/AzureFunctionExamples" target="_blank">AzureFunctionExample github repository</a> with a solution that can be run local and be published to the cloud. An Azure Function v1 (Net Framework project) and a v2 (Net Standard and still in preview) example respond to a queue message, get some extra data from a database using Entity Framework Core, retrieve an emailtemplate from a blob storageaccount and then send an email using the data and the template with the <a href="https://sendgrid.com/solutions/email-api/" target="_blank">SendGrid emailservice</a>. (SendGrid offers an emailservice API that is free to use up to 100 emails per day)
</p>
<p>
	I used a separate class library to demonstrate that referencing other project libraries works the same as other project types. Complex Azure Functions can be build as a small microservice with a layer architecture to separate your execution/workflow, business logic and resource access into separate libraries.
</p>
<p>
 	In my example repository I used and referenced a .Net Standard 2.0 library by both an Azure Function v1 and v2 project but you can also add a class library matching the framework version of the Azure Function project. The code executed by the Functions is the same, just the underlying Framework and Azure Function runtime differs. Note that when installing NuGet package on a project it may reference different dll versions and dependencies with the same NuGet package version for the different Standard, Core and Framework projects. So when the referenced dependencies on the class library and executing project do not match it may result in runtime errors because expected DLL versions do not match on the executing project. An example of this is the WindowsAzure.Storage package version as shown below.
 <br/><img src="{{"/assets/images/20180608/AzureStoragePackageVersionDifferences.png" | relative_url }}" alt="AzureStorage Package Version Differences"/>
 </p>
<h3>Set up and run the storage emulator</h3>
<p> 
	In the windows programs or start menu find and start the Azure Storage Emulator. I encountered some issues starting the Azure Storage Emulator using the default LocalDB instance due to path naming and/or access rights, however I did manage to set up the needed database on my local SQLExpress instance. To force the Azure Storage Emulator to initialize on a named SQL instance instead of the default LocalDB use the command below and replace SQLExpress with the name of the SQL Server instance that you wish to use: 
</p>
{% highlight shell %}
AzureStorageEmulator.exe init /server SQLEXPRESS
{% endhighlight %} 
<p>
	After it's started, an icon for the emulator will pop-up. Use the context menu on the icon to use the "Show Storage Emulator UI" to view the command prompt with the possible commands for the emulator below.
	<br/><img src="{{"/assets/images/20180608/AzureStorageEmulatorCommandPromt.png" | relative_url }}" alt="Azure Storage Emulator Commandpromt"/>
</p>
<p>
	You can explore the contents of the endpoints using the Visual Studio Cloud Explorer window (found under the View main menu in Visual Studio) under the (Local) and then Storage Accounts node.
	You can access the storage programmatically using the account name and key found on the <a href="https://docs.microsoft.com/nl-nl/azure/storage/common/storage-use-emulator" target="_blank">Microsoft Docs page</a> for the emulator. The access key and account are defaults and publicly known thus they are also included in the config and appsettings files in the projects of my <a href="https://github.com/SamanthaNeilen/AzureFunctionExamples" target="_blank">AzureFunctionExample GitHub repository</a>. 
</p>
<p>
	Note that the Storage Emulator will shutdown when the computer is turned of and will need to be restarted to access it after rebooting your computer.
</p>
<h3>Create a simple NetCore console application to post a queue message</h3>
<p>
	After the Azure Storage Emulator is running you can create a simple console application to post a message to the local queue storage. First create a new .Net Core Console application.	
	Next add an appsettings.json file to store any credentials and settings. Fill it with the content shown below and be sure to mark the file as "Copy to Output Directory" with a value of always in the file properties window.
</p>
{% highlight json %}
{
  "AzureStorageUri": "http://127.0.0.1:10001/devstoreaccount1",
  "AzureStorageAccountName": "devstoreaccount1",
  "AzureStorageAccountKey": "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==",
  "AzureStorageQueueName": "messages"
}
{% endhighlight %}
<p>
	Next create a class that will be our data structure. We can use this to serialize easily to JSON. Because an Azure Function can automatically convert our serialized json queue message back to an object as I will show in the later part of this post where we create the actual Azure Function.
</p>
{% highlight c# %}
internal class QueueMessage
{
	public int Person { get; set; }
	public int Order { get; set; }
	public int Status { get; set; }
}
{% endhighlight %}
<p>
	Add the following NuGet packages to your project:<br/>
- Microsoft.Extensions.Configuration.Json, used to access the appsettings file<br/>
- WindowsAzure.Storage, used to access the queue storage<br/>
<p>
	The code below shows the program.cs that can create and post a message.
</p>
{% highlight c# %}
using System;
using System.IO;
using Microsoft.Extensions.Configuration;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage.Queue;
using Newtonsoft.Json;

namespace QueueMessageClient
{
    class Program
    {
        static void Main(string[] args)
        {
            //Get access to appsettings
            var builder = new ConfigurationBuilder()
               .SetBasePath(Directory.GetCurrentDirectory())
               .AddJsonFile("appsettings.json");
            var configuration = builder.Build();

            //Get or create queue in StorageAccount
            var client = new CloudQueueClient(new Uri(configuration["AzureStorageUri"]), 
                new StorageCredentials(configuration["AzureStorageAccountName"],configuration["AzureStorageAccountKey"]));
            var queue = client.GetQueueReference(configuration["AzureStorageQueueName"]);
            var creationresult = queue.CreateIfNotExistsAsync().Result;

            //Create and send message as JSON
            var message = new QueueMessage
            {
                Order = 1,
                Person = 1,
                Status = (int)OrderStatus.Processed
            };
            var messageJSON = JsonConvert.SerializeObject(message);
            queue.AddMessageAsync(new CloudQueueMessage(messageJSON)).Wait();

            Console.WriteLine("Message Sent");
            Console.ReadKey();
        }
    }
}
{% endhighlight %}
<p>
	Use the Cloud explorer to check if your message has been created in the queue.
	<br/><img src="{{"/assets/images/20180608/CloudExplorerAndQueueWindows.png" | relative_url }}" alt="Cloud Explorer and Queue windows"/>
</p>
<p>
	To post a message to an actual Azure Blob Storage, simply replace the access credentials and storage uri to values associated to an existing Azure Blob Storage.
</p>
<h3>Create a local database</h3>
<p>
	I've added a simple database project to my example solution. Create a local database in your SQL Instance, do a Schema Compare and create the tables and then run the testdata script included for a few records of data.
	(For more information on these steps outlined see my <a href="https://samanthaneilen.github.io/2017/11/24/managing-a-sql-server-database-from-a-visual-studio-database-project.html" target="_blank">previous blog post</a> on using a database project.
</p>
<h3>Create an Azure Function project</h3>
<p>
	The Azure Function project can be found under the cloud category (the Azure workload needs to be installed). When you create a project you immediately get a popup asking what kind of function you want to create within the project.
	<br/><img src="{{"/assets/images/20180608/AzureFunctionProject.png" | relative_url }}" alt="Azure Function Project"/>
</p>
<p>
	Notice the popup for the version of your function. Azure Functions V1 is based on the full .Net Framework and currently supports more triggers and bindings than the Azure Functions V2 that is currently still in preview and is based on the .Net Standard Framework. The choices for the kind of Azure Function to create with the project are actually limited to the most common ones. If you do not see the trigger you want to use, select the empty option and use the File then New option in the project context menu in the Solution Explorer to add a new Azure Function.	
</p>
<p>
Function V1 default trigger dialog:
<br/><img src="{{"/assets/images/20180608/CreateV1Function.png" | relative_url }}" alt="Create V1 Function"/>
</p>
<p>
Function V2 default trigger dialog:
<br/><img src="{{"/assets/images/20180608/CreateV2Function.png" | relative_url }}" alt="Create V2 Function"/>
</p>
<p> 
	When using the Add then Add New Item option in the context menu of the project, you get a lot more options for the Azure Function that you want to create.
</p>
<p>
    Add New Item dialog:
    <br/><img src="{{"/assets/images/20180608/AzureFunctionItem.png" | relative_url }}" alt="Azure Function Item"/>
</p>
<p>
	Azure Function version 1 triggers:
	<br/><img src="{{"/assets/images/20180608/AzureFunctionTypesV1.png" | relative_url }}" alt="Azure Function Types V1"/>
</p>
<p>
	Azure Function version 2 triggers:
	<br/><img src="{{"/assets/images/20180608/AzureFunctionTypesV2.png" | relative_url }}" alt="Azure Function Types V2"/>
</p>
<p>In my example project I created a Queue Function. For the settings I used the UseDevelopment storage and chose AzureWebJobsStorage for my connectionstring. A default project contains the Function1.cs with the default input variables defined by the chosen function template. More information on the bindings can be found <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings" target="_blank">here</a>.
</p>
<p>Default content of Function1.cs for a queue trigger</p>
{% highlight c# %}
public static class Function1
{
	[FunctionName("Function1")]
	public static void Run([QueueTrigger("messages", Connection = "AzureWebJobsStorage")]string myQueueItem, TraceWriter log)
	{
		log.Info($"C# Queue trigger function processed: {myQueueItem}");
	}
}
{% endhighlight %}
<p>
Other than the Function1.cs there is a local.settings.json and a host.json. The local.settings.json is your appsettings file for running locally. When deployed to Azure the Azure Function will use the appsettings defined in the Application Settings page of the Function App in the Azure Portal. 
</p>
{% highlight json %}
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "AzureWebJobsDashboard": "UseDevelopmentStorage=true"
    }
}
{% endhighlight %}
<p>
The host.json is used to defined certain runtime settings. Possible settings and the defaults if no settings are defined can be found <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json" target="_blank">here </a>. The most important setting in the host.json is the functionTimeout setting. Azure Functions that run on a consumption plan only have a timeout of 5 minutes, meaning that the execution is terminated after 5 minutes without any decent error. When persisting state within the function this may lead to  leaving data in an unfinished or corrupted state. In a queue function a  message is set invisible when processing and only removed from the queue after being fully processed. When processing is interrupted without removing the message, the message will become visible again after a configurable timeout. The message then triggers another run of the Azure Function. This can trigger an infinite loop of failing requests in the message never finishes processing within the functionTimeout interval. On a consumption (pay-as-you-go plan) you can currently set the maximum to a 10 minute timeout. If you have workloads that need more processing time you should switch to an app service billing plan and be sure to remove the functionTimeout setting from the host.json to avoid functions being terminated.
</p>
{% highlight json %}
{
	"functionTimout": "00:10:00"
}
{% endhighlight %}
<p>
The next thing of interest is the NuGet package for the Azure Function SDK called Microsoft.NET.Sdk.Functions. 
<br/><img src="{{"/assets/images/20180608/NuGetSdkFunctions.png" | relative_url }}" alt="NuGet Sdk Functions"/>
<br/>
This NuGet contains al the references that are available in the Azure Function Environment. A thing to remember when using functionality, like accessing azure storage, is to use the version referenced in the SDK package because the Azure Function Apps runtime does not run the latest versions of packages like AzureStorage or SendGrid. When creating separate referenced class libraries use the same version Azure Function SDK to avoid mismatching dependencies. You will get a runtime error stating that the dependent dll with the expected version number is not found.
</p>
<p>
Now that we have an Azure Function that can respond to a queue message and that logs a logmessage. Run the project, you can add a breakpoint to the log line to see that debugging works as expected. Azure Functions projects start a console with logging as shown below.
<p>
	Azure Function v1 run in the console window:
	<br/><img src="{{"/assets/images/20180608/ConsoleRunningFunctionV1.png" | relative_url }}" alt="Console Running Function V1"/>
</p>
<p>
	Azure Function v2 run in the console window:
	<br/><img src="{{"/assets/images/20180608/ConsoleRunningFunctionV2.png" | relative_url }}" alt="Console Running Function V1"/>
</p>
</p>
Debugging one of the Functions:
<br/><img src="{{"/assets/images/20180608/DebugDefaultAzureFunction.png" | relative_url }}" alt="DebugDefaultAzureFunction"/>
</p>
<p>When working with queue triggers be aware that only one Function can respond to a queue message. If multiple functions are listening to the same queue only the first to respond to the message (and set the message invisible) will execute.</p>
<p>
There is one known issue when running a V2 Azure Function in that it has trouble finding the Azure CLI Runtime located in you Users/AppData/ folder when the path (usually your usename) contains spaces. Introduce a launchSettings.json file inside a properties folder in your project directory to help the runtime out in locating the correct location for the needed assemblies as described <a href="https://github.com/Azure/Azure-Functions/issues/623" target="_blank">here</a>.
</p>
{% highlight json %}
{
  "profiles": {
    "MyApp": {
      "commandName": "Executable",
      "executablePath": "C:\\Program Files\\dotnet\\dotnet.exe",
      "commandLineArgs": "\"C:\\Users\\XXXX\\AppData\\Roaming\\npm\\node_modules\\azure-functions-core-tools\\bin\\Azure.Functions.Cli.dll\" host start",
      "workingDirectory": "$(TargetDir)"
    }
  }
{% endhighlight %}
<p>
When running and debugging the message you can see that the message input is currently a string that needs to be deserialised. Since the structure is a known JSON input you can just add a class defining this structure to your Function project or a referenced class library and replace string type input with the type of your class.
</p>
{% highlight c# %}
public class QueueMessage
{
	public int Person { get; set; }
	public int Order { get; set; }
	public int Status { get; set; }
}
{% endhighlight %}
{% highlight c# %}
[FunctionName("ProcessMessageV1")]
        public static void Run([QueueTrigger("messages", Connection = "AzureWebJobsStorage")]QueueMessage myQueueItem, TraceWriter log)
{% endhighlight %}
<p>
After doing this you can see in the debugger that you get a nice object input ready for use in your function code. 
<br/><img src="{{"/assets/images/20180608/DebugSerializedInput.png" | relative_url }}" alt="Debug Serialized Input"/>	
</p>
<h3>Access the database using Entity Framework Core</h3>
<p>
For the data access framework I will be using Entity Framework Core. Add the Microsoft.EntityFrameworkCore.SqlServer NuGet package to your Function project or a referenced class library and specify a database context as shown below:
</p>
<p>Person class:
{% highlight c# %}
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace AzureFunctionExample.ResourceAccess.DataAccess
{
    public class Person
    {
        public Person()
        {
            Orders = new HashSet<Order>();
        }

        [Key]
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string Email { get; set; }
        public virtual ICollection<Order> Orders { get; set; }
    }
}
{% endhighlight %}
</p>
<p>Order class:
{% highlight c# %}
using System.ComponentModel.DataAnnotations;

namespace AzureFunctionExample.ResourceAccess.DataAccess
{
    public class Order
    {
        [Key]
        public int Id { get; set; }
        public int PersonID { get; set; }        
        public string OrderNumber { get; set; }
        public int Status { get; set; }
        public virtual Person Person { get; set; }
    }
}

{% endhighlight %}
</p>
<p>Context class:
{% highlight c# %}
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;

namespace AzureFunctionExample.ResourceAccess.DataAccess
{
    public class OrderDbContext : DbContext
    {
        public OrderDbContext(DbContextOptions<OrderDbContext> options) : base(options)
        {
        }

        public DbSet<Person> Person { get; set; }
        public DbSet<Order> Order { get; set; }


        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Person>()
               .HasMany(o => o.Orders)
               .WithOne(p => p.Person)
               .HasForeignKey(o => o.PersonID);
        }

        public static OrderDbContext GetInstance(string connectionString)
        {
            var builder = new DbContextOptionsBuilder<OrderDbContext>().UseSqlServer(connectionString);
            return new OrderDbContext(builder.Options);
        }
    }
}
{% endhighlight %}
</p>
<p>
Next add a connectionstring to your local.settings.json referencing the database you created with the database project.
</p>
{% highlight json %}
"OrderDbConnection": "Server=.\\SQLEXPRESS;Database=OrderDb;Integrated Security=True;"
{% endhighlight %}
<p>
Then add the code to your Azure Function to create and call the context. The code snippet below shows how to get the ordernumber and person name into a MailSettings object that can be used as input for a mail sending service class.
</p>
{% highlight c# %}
public class MailData
{
	public string PersonName { get; set; }
	public string Email { get; set; }
	public string OrderNumber { get; set; }
	public string Status { get; set; }
	public string EmailTemplate { get; set; }
}
{% endhighlight %}
{% highlight c# %}
[FunctionName("ProcessMessageV1")]
public static void Run([QueueTrigger("messages", Connection = "AzureWebJobsStorage")]QueueMessage myQueueItem, TraceWriter log)
{
	var dbContext = OrderDbContext.GetInstance(Environment.GetEnvironmentVariable("OrderDbConnection"));
	var order = dbContext.Order.Include(o => o.Person).FirstOrDefault(o => o.Id == myQueueItem.Order && o.PersonID == myQueueItem.Order);

	var mailData = new MailData
	{
		PersonName = $"{order.Person.FirstName} {order.Person.LastName}",
		Email = order.Person.Email,
		OrderNumber = order.OrderNumber,
		Status = ((OrderStatus)myQueueItem.Status).ToString(),
		EmailTemplate = blobService.GetEmailTemplateContents(Environment.GetEnvironmentVariable("EmailTemplateContainerName"),
			Environment.GetEnvironmentVariable("EmailTemplateBlobName"))
	};

	log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
{% endhighlight %}
<p>
Debug or log the contents of the MailSettings object instance to check if the database data is retrieved as expected.
</p> 
<h3>Retrieve an emailtemplate file from a blobstorage</h3>
<p>
First create a emailtemplate html as shown below and upload it to an emaltemplate container in your local development blob storage. The text between brackets will be replaced with the data we retrieved from the database before sending the email.
</p>
{% highlight html %}
<html xmlns="http://www.w3.org/1999/xhtml">
<head><title></title></head>
<body>
    <p>Dear {CONTACT},</p>
    <p>You order with ordernumber {ORDERNUMBER} now has status {STATUS}.</p>
    <p>Regards,<br/>
        Sender
    </p>
</body>
</html>
{% endhighlight %}
<p> 
Next add the Microsoft.NET.Sdk.Functions NuGet package to your Function project or class library so we have a reference to the WindowsAzureStorage. 
Below is the code to get a reference to a blob storage, get the file as a stream and then read the stream from the beginning to get the contents of a blob as a string. 
</p>
{% highlight c# %}
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage.Blob;
using System;
using System.IO;

namespace AzureFunctionExample.ResourceAccess
{
    public class BlobStorageService
    {
        private readonly string _storageUri;
        private readonly string _accountName;
        private readonly string _key;

        public BlobStorageService(string storageUri, string accountName, string key)
        {
            _storageUri = storageUri;
            _accountName = accountName;
            _key = key;
        }

        public string GetEmailTemplateContents(string containerName, string emailTemplateName)
        {
            var result = string.Empty;
            CloudBlobClient client = new CloudBlobClient(new Uri(_storageUri), new StorageCredentials(_accountName, _key));
            var container = client.GetContainerReference(containerName);

            if (container.ExistsAsync().Result)
            {
                using (var stream = new MemoryStream())
                {
                    var blob = container.GetBlobReference(emailTemplateName);
                    blob.DownloadToStreamAsync(stream).Wait();

                    stream.Position = 0; 
                    result = new StreamReader(stream).ReadToEnd();
                }
            }
            else
            {
                throw new Exception($"Container {containerName} not found");
            }
            return result;
        }        
    }
}
{% endhighlight %}
<p>
All of the parameters for the constructor en method will be defined in the local.settings.json of the calling assembly so we can easily change the location and name of the emailtemplate without the need to redeploy the function. The settings below show the connection to the local Azure Emulator blob storage.
</p>
{% highlight json %}
"BlobStorageUri": "http://127.0.0.1:10000/devstoreaccount1",
"BlobStorageAccount": "devstoreaccount1",
"BlobStorageKey": "Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==",
"EmailTemplateContainerName": "emailtemplates",
"EmailTemplateBlobName": "emailtemplate.html"
{% endhighlight %}
<p>
Next we can create the blobservice in our function instance and call the method to get the emailtemplate contents as a string.
</p>
{% highlight c# %}
[FunctionName("ProcessMessageV1")]
public static void Run([QueueTrigger("messages", Connection = "AzureWebJobsStorage")]QueueMessage myQueueItem, TraceWriter log)
{
    var dbContext = OrderDbContext.GetInstance(Environment.GetEnvironmentVariable("OrderDbConnection"));
    var blobService = new BlobStorageService(Environment.GetEnvironmentVariable("BlobStorageUri"),
        Environment.GetEnvironmentVariable("BlobStorageAccount"),
        Environment.GetEnvironmentVariable("BlobStorageKey")
        );

    var order = dbContext.Order.Include(o => o.Person).FirstOrDefault(o => o.Id == myQueueItem.Order && o.PersonID == myQueueItem.Order);

    var mailData = new MailData
    {
        PersonName = $"{order.Person.FirstName} {order.Person.LastName}",
        Email = order.Person.Email,
        OrderNumber = order.OrderNumber,
        Status = ((OrderStatus)myQueueItem.Status).ToString(),
        EmailTemplate = blobService.GetEmailTemplateContents(Environment.GetEnvironmentVariable("EmailTemplateContainerName"),
            Environment.GetEnvironmentVariable("EmailTemplateBlobName"))
    };

    log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
{% endhighlight %}
<p>
When calling this code from the Azure Function V1 (Framework project), referencing a .Net Standard 2.0 library you will get a runtime error because as stated before referenced dll versions in the referenced SDK NuGet package do not match.
<br/><img src="{{"/assets/images/20180608/AzureStorageDLLVersionError.png" | relative_url }}" alt="AzureStorageDLLVersionError"/>
<br/><img src="{{"/assets/images/20180608/AzureStoragePackageVersionDifferences.png" | relative_url }}" alt="AzureStorage Package Version Differences"/>
<br/>
To resolve this issue add the matching Windows Azure Storage version NuGet package to the Azure Function V1 project. After that it will have the proper version in the output directory of the Azure Function Project.
</p>
<br/><img src="{{"/assets/images/20180608/ExtraWindowsAzureStoragePackage.png" | relative_url }}" alt="Extra Windows Azure Storage Package"/>
<p>
Again you can use the console or debugger to verify that the retrieved string matches the contents of the uploaded file. 
</p>
<h3>Send an email using the SendGrid email client</h3>
<p>
Now that we have all the settings data and a html string with placeholders for the data, we can add a sendemailservice to our project. (There is also an output binding that can send an email using a Azure SendGrid resource as explained <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-sendgrid" target="_blank">here</a>) but I prefer creating a custom serviceclient because you can change the implementation to use another online emailing service if needed and have more control for error handling.
</p>
<p>
First add the SendGrid NuGet package. Then add EmailService as show below. The key passed in the constructor is a unique API key that you can create in your SendGrid account under the Settings then API Keys option.
</p>
{% highlight c# %}
using AzureFunctionsExample.Shared;
using SendGrid;
using SendGrid.Helpers.Mail;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;

namespace AzureFunctionExample.ResourceAccess
{
    public class EmailService
    {
        private readonly string _apiKey;
        private readonly string _from;
        private IEnumerable<HttpStatusCode> _validStatusCodes = new List<HttpStatusCode>{
            HttpStatusCode.Accepted,
            HttpStatusCode.OK
        };


        public EmailService(string apiKey, string from)
        {
            _apiKey = apiKey;
            _from = from;
        }


        public void SendEmail(MailData mailData)
        {
            var message = new SendGridMessage() {
                From = new EmailAddress(_from)
            };

            message.AddTo(new EmailAddress(mailData.Email));

            message.HtmlContent = mailData.EmailTemplate
                .Replace("{CONTACT}",mailData.PersonName)
                .Replace("{ORDERNUMBER}", mailData.PersonName)
                .Replace("{STATUS}", mailData.Status);
                
            message.Subject = "Mail from Azure Function";

            var client = new SendGridClient(_apiKey);
            var response = client.SendEmailAsync(message).Result;
            if (!_validStatusCodes.Contains(response.StatusCode))
            {
                throw new Exception("Error sending mail");
            }
        }
    }
}

{% endhighlight %}
<p>
Next we add the API key and from email settings to the local.settings.json file and then create the mailservice calls the with the data in the Function. 
</p>
{% highlight json %}
"SendGridApiKey": "YOUR_KEY_HERE",
"FromEmail": "YOUR_SENDER_EMAIL_HERE",
{% endhighlight %}
{% highlight c# %}
[FunctionName("ProcessMessageV1")]
public static void Run([QueueTrigger("messages", Connection = "AzureWebJobsStorage")]QueueMessage myQueueItem, TraceWriter log)
{
    var dbContext = OrderDbContext.GetInstance(Environment.GetEnvironmentVariable("OrderDbConnection"));
    var blobService = new BlobStorageService(Environment.GetEnvironmentVariable("BlobStorageUri"),
        Environment.GetEnvironmentVariable("BlobStorageAccount"),
        Environment.GetEnvironmentVariable("BlobStorageKey")
        );

    var order = dbContext.Order.Include(o => o.Person).FirstOrDefault(o => o.Id == myQueueItem.Order && o.PersonID == myQueueItem.Order);

    var mailData = new MailData
    {
        PersonName = $"{order.Person.FirstName} {order.Person.LastName}",
        Email = order.Person.Email,
        OrderNumber = order.OrderNumber,
        Status = ((OrderStatus)myQueueItem.Status).ToString(),
        EmailTemplate = blobService.GetEmailTemplateContents(Environment.GetEnvironmentVariable("EmailTemplateContainerName"),
            Environment.GetEnvironmentVariable("EmailTemplateBlobName"))
    };

    var senGridService = new EmailService(Environment.GetEnvironmentVariable("SendGridApiKey"), Environment.GetEnvironmentVariable("FromEmail"));
    senGridService.SendEmail(mailData);

    log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
{% endhighlight %}
<p>
Next run the function to send an email with your SendGrid account. (Some servers or domain names may have the SendGrid domain blacklisted and deny delivery but you can verify mails send in your SendGrid account under the Activity setting) 
</p>
<h3>Some notes on publishing to Azure</h3>
<p>Sometimes a publish to a new Azure Function app fails because the Function app resource is not created fast enough. Usually a second publish will succeed.</p>
<p>You cannot publish a version 1 Azure Function to a version 2 Azure Function app and publishing an Azure Function v2 app to an existing v1 app will cause a pop-up asking if you want to upgrade the version.</p>
<p>Publishing will not create the appropriate Application Setting keys for the Azure Function app in Azure. You will have to create them separately in the Azure portal.</p>



