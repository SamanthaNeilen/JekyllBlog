---
layout: post
title:  "C# script Azure Functions example"
date:   2018-07-14 00:00:00 +0100
tags: Azure AzureFunctions
---
<p>
C# Script Functions are another way to create Azure Functions based on C#. These functions rely on C# script files (.csx) that are not compiled into a DLL. There is no Visual Studio project to create or manage the files. You use command line tooling for development and the advantage is that you can directly edit and run your code in the Azure portal. Script Functions can be developed and tested locally before uploading the Function to the portal.
</p>
<p>
If you would like to know more about compiled Azure Functions using the Visual Studio Azure Function project see <a href="https://samanthaneilen.github.io/2018/06/08/pre-compiled-azure-functions-example.html" target="_blank">my last blog post about pre-compiled Azure Functions</a>.
</p>
<p>
All the code from the following example can also be found in my <a href="https://github.com/SamanthaNeilen/AzureFunctionExamples" target="_blank">AzureFunctionExample repository</a> in the solution folder AzureFunction_CsharpScript.
</p>

**Table of contents:**
* Table of Contents
{:toc}
### Create a C# script Azure Function
<p>
Install npm and Node.Js from the <a href="https://nodejs.org/en/" target="_blank">Node.Js website</a>.
</p>

Install npm Azure Functions core tools using the command line. I used the v1 (.NET Framework version) using the command below. 
{% highlight shell %}
npm install -g azure-functions-core-tools
{% endhighlight %}

<p>
Append the package name with @core to install de v2 version. The v2 version is built on .NET Core and is currently still in preview (the same as with the compiled Azure Functions)
</p>
<p>
The files for the azure-function-core-tools are placed in %USER%\AppData\Roaming\npm on your disk.
In the npm\node_modules\azure-functions-core-tools\bin folder, you can view all the Framework and Nuget package DLLs available for use at runtime.
</p>
<p>
If you wish to check the version of the downloaded package and the commands for the command line use the command below. For command usage, see the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local" target="blank">Microsoft Docs guidance on developing C# script Functions</a>.
</p>

{% highlight shell %}
func version
{% endhighlight %}
<p>
Next, I want to create a queue Function based on the C# language. Go to the directory where you want to create the function folder. And use the command below.
</p>

{% highlight shell %}
func init AzureFunctionsExample.ScriptV1
{% endhighlight %}
<br/><img src="{{"/assets/images/20180714/cmd.exe-func-init.png" | relative_url }}" alt="cmd.exe func init"/>
<p>
This command will create a “project folder” for an empty Azure Function project with the host and localsettings JSON files. (These are used the same as for compiled Azure Functions. See <a href="https://samanthaneilen.github.io/2018/06/08/pre-compiled-azure-functions-example.html" target="_blank">my previous blogpost about compiled Azure Functions</a> or the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local#local-settings-file" target="blank">Microsoft Docs pages</a> for more information) 
</p>
<p>
Also, note the .vscode folder. Seeing as there is no .csproj file, the support for regular Visual Studio is limited. You can manage in the files in Visual Studio by creating a class library or solution folders and importing the files but with Visual Studio Code, you can just open the folder to manage the files. 
</p>
<p>
Visual Studio Code also has a plugin for Azure Functions that adds an Azure tab to your project. It also has options to create Azure Functions but these will create compiled Functions. When opening the folder you may be prompted to optimize the Function but as it was not created using the plugin, it will not optimize. 
</p>
<p>
I used regular Visual Studio with a class library and the regular command-line shell for all the next steps because I did not have time to look into debugging script Azure Functions with Visual Studio Code. NPM installs a 32-bit version of the CLI and Visual Studio Code does not seem to support debugging the 32-bit process.
</p>
<p>
The command below is used to create the files for the actual Function in a subfolder of the Azure Function app folder.
</p>

{% highlight shell %}
func new
{% endhighlight %}
<p>
Next, navigate the selection options to create a C# Queue trigger and provide a name for the Function.
<br/><img src="{{"/assets/images/20180714/cmd.exe-func-new.png" | relative_url }}" alt="cmd.exe func new"/>
</p>
<p>
This will create a new subfolder with a function.json that describes the input/output bindings, a run.csx that has the “main” method for the function called Run and a sample.dat file that can be used later to provide message data while debugging. It also has a readme file with little information. I just deleted the readme file. 
</p>
<p>Function.json:</p>

{% highlight json %}
{
  "disabled": false,
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "myqueue-items",
      "connection": ""
    }
  ]
}
{% endhighlight %}
<p>Run.csx</p>
{% highlight C# %}
using System;

public static void Run(string myQueueItem, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
{% endhighlight %}
<p>
For easy management just include all the files inside your Visual Studio class library project so you can access and manage them from the Solution Explorer as shown in the image below.
<br/><img src="{{"/assets/images/20180714/solutionexplorer-AzureFunctionsExample.ScriptV1.png" | relative_url }}" alt="Solution Explorer AzureFunctionsExample.ScriptV1"/>
</p>
<p>
Now we have all the files needed for the Function but first, we need to configure the connection to the Azure Storage emulator before we can start it up. 
</p>
<p>
Go into local.settings.json in the root folder and change the Values node with the content below to configure the storage connections:
</p>

{% highlight json %}
"Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "AzureWebJobsDashboard": "UseDevelopmentStorage=true"
  }
{% endhighlight %}
<p>
Next, go to the function.json and set the connection to the AzureWebJobsStorage and the queueName to messages. If you do not have the Azure Storage Emulator set up yet view the Azure Storage Emulator paragraph in <a href="https://samanthaneilen.github.io/2018/06/08/pre-compiled-azure-functions-example.html" target="_blank">my post about compiled Azure Functions</a>. 
</p>

{% highlight json %}
{
  "disabled": false,
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "messages",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
{% endhighlight %}
<p>
We can now run the Function by using the command shown below in the command shell in the root folder (where the host.json file is). Make sure that the Azure Storage Emulator is running,
</p>
{% highlight shell %}
func host start --debug VS
{% endhighlight%}
<br/><img src="{{"/assets/images/20180714/cmd.exe-func-host-start.png" | relative_url }}" alt="cmd.exe func host start"/>
<p>
This will start the Azure Function the same way that it did with the compiled Azure Function.  
</p>
<p>
When specifying debug mode you also get the JIT Debugger window. Just choose the Visual Studio instance where the run.csx is available to attach the debugger.
<br/><img src="{{"/assets/images/20180714/jit-debug-window.png" | relative_url }}" alt="jit debug window"/>
</p>
<p>
You will also get the application in the break mode tab in your Visual Studio instance. Just click the continue execution option. (You may have to press enter in the command shell to continue loading of the Function if it was interrupted by the debug prompt before it will pick up any messages)
</p>
<p>
Next set a breakpoint in the run.csx and use the QueueMessage project from my <a href="https://github.com/SamanthaNeilen/AzureFunctionExamples" target="_blank">AzureFunctionExamples</a> repository or any other method to post a message to the queue and see the debugger in the run.asxc.
<br/><img src="{{"/assets/images/20180714/debug-run.csx.png" | relative_url }}" alt="debug run.csx"/>
</p>
<p>
For extra methods and simple Azure Functions, you could create all extra classes and methods in the run.csx file. For better maintainability create classes and extra code into separate C# script (.csx) files or in a separate class library compiled to a DLL. You can also reference the .NET assemblies that are available for the Azure Function runtime. For more information see the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp" target="_blank"> Microsoft Docs on C# script</a>.
</p>
<p>
In my <a href="https://github.com/SamanthaNeilen/AzureFunctionExamples" target="_blank">AzureExampleFunctions repository</a>. I’ve created a QueueMessage.csx class used to convert the input queue message from a string to an object instance. First, add a QueueMessage.csx file to the ProcessMessageScriptV1 Function folder with the definition of the QueueMessage class (as shown below). There are no using or namespace definitions in the file. Next use the #load “QueueMessage.csx” statement in the run.csx to complete the reference binding and change the type for myQueueItem from string to QueueMessage. The code should look as below:
</p>
<p>
QueueMessage.csx:
</p>

{% highlight C#%}

internal class QueueMessage
{
    public int Person { get; set; }
    public int Order { get; set; }
    public int Status { get; set; }
}

internal enum OrderStatus
{
    Concept = 0,
    Processing = 1,
    Processed = 2
}
{% endhighlight %}
<p>run.csx:</p>
{% highlight C# %}
#load "QueueMessage.csx"

using System;

public static void Run(QueueMessage myQueueItem, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
{% endhighlight %}
<p>
Next run the Function to see that the myQueueItem is successfully converted from a JSON message to a QueueMessage instance.
<br/><img src="{{"/assets/images/20180714/debug-run.csx-queuemessage-object.png" | relative_url }}" alt="debug run.csx queuemessage object"/>
</p>
<p>
Extra .csx files can also be placed in subfolders. If QueueMessage.csx was placed in a subfolder called helpers you would use the #load “helpers/QueueMessage.csx” in the run.csx file. The #load statement contains a relative path to the file. See the <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp#reusing-csx-code" target="_blank"> Microsoft Docs on C# script</a> for an example.
</p>

### Referencing a custom DLL in an Azure Script Function
<p>
Next, I can create separate DLL containing a Data Access Layer and reference it in my script Function. 
(I couldn’t use the AzureFunctionExample.ResourceAccess library already present in the solution because I got exceptions that EntityFrameworkCore was not supported on the platform). Just add a .NET Framework library with the Entity Framework package and define the data access classes below:
</p>
<p>Order class:</p>

{% highlight C# %}
using System.ComponentModel.DataAnnotations;

namespace AzureFunctionsExample.ScriptResources.DataAccess
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
<p>Person class:</p>
{% highlight C# %}
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace AzureFunctionsExample.ScriptResources.DataAccess
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
<p>OrderDbContext class:</p>
{% highlight C# %}
using System.Data.Entity;
using System.Data.Entity.ModelConfiguration.Conventions;

namespace AzureFunctionsExample.ScriptResources.DataAccess
{
    public class OrderDbContext : DbContext
    {
        public OrderDbContext(string connectionString) : base(connectionString)
        {
            Database.SetInitializer<OrderDbContext>(null);
        }
        
        public DbSet<Person> Person { get; set; }
        public DbSet<Order> Order { get; set; }
     
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Person>()
               .HasMany(o => o.Orders)
               .WithRequired(p => p.Person)
               .HasForeignKey(o => o.PersonID);
     
            modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
     
            base.OnModelCreating(modelBuilder);
        }
    }
}
{% endhighlight%}
<p>
To use this library in my Function I need to get the output DLL and the extra references to a location accessible to the script Function. So first I copy all the DLL from the bin output to an empty references folder using a post-build event.
</p>

{% highlight shell %}
xcopy /Y/S $(TargetDir)$(ProjectName).dll $(SolutionDir)AzureFunctionsExample.ScriptV1\ProcessMessageScriptV1\references
xcopy /Y/S $(TargetDir)EntityFramework.dll $(SolutionDir)AzureFunctionsExample.ScriptV1\ProcessMessageScriptV1\references
xcopy /Y/S $(TargetDir)EntityFramework.SqlServer.dll $(SolutionDir)AzureFunctionsExample.ScriptV1\ProcessMessageScriptV1\references
{% endhighlight %}
<br/><img src="{{"/assets/images/20180714/project-properties-xcopy-build-event.png" | relative_url }}" alt="project properties xcopy build event"/>
<p>
This will ensure that after every build all the necessary DLL will be in that folder (even though I excluded that folder from source control with the .gitignore file) 
</p>
<p>
Next, add all the necessary settings to the local.settings.json. so it looks like below:
</p>

{% highlight json %}
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "AzureWebJobsDashboard": "UseDevelopmentStorage=true",
    "OrderDbConnection": "Server=.\\SQLEXPRESS;Database=OrderDb;Integrated Security=True;",
  }
}
{% endhighlight %}
<p>
And finally, add the reference and using statements to the run.csv and change the implementation of the run.csx to the code shown below:
</p>

{% highlight C# %}
#load "QueueMessage.csx"
#r "references/EntityFramework.dll"
#r "references/AzureFunctionsExample.ScriptResources.dll"
#r "netstandard"

using AzureFunctionsExample.ScriptResources.DataAccess;
using System.Data.Entity;
using System;
using System.Linq;

public static void Run(QueueMessage myQueueItem, TraceWriter log)
{
    var dbContext = new OrderDbContext(Environment.GetEnvironmentVariable("OrderDbConnection"));
    var order = dbContext.Order.Include(o => o.Person).FirstOrDefault(o => o.Id == myQueueItem.Order && o.PersonID == myQueueItem.Person);

    log.Info($"{order.Person.FirstName} {order.Person.LastName}");
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
{% endhighlight %}
<p>
By referencing DLL or just using .csx files you can create pretty complex C# script Functions with the same functionality as a pre-compiled Azure Function.
</p>
### Run and test in the Azure Portal
<p>
The command line offers a login and publish-command to push your local Function app to a hosted Azure Function App. Another way you can upload the files is via de App Service Editor under the Platform Settings for an Azure App Function in the Azure Portal. Remember to copy the key-value pairs from the appsettings.json to the Application Settings page of the Azure Function App in the portal.
</p>
<p>
With script Functions, you can view the files and edit and run the Function directly in the portal with the editor shown below.
<br/><img src="{{"/assets/images/20180714/azure-function-app-editor.png" | relative_url }}" alt="azure-function-app-editor"/>
</p>


