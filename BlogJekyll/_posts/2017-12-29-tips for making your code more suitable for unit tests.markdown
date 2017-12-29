---
layout: post
title:  "Tips for making your code more suitable for unit testing"
date:   2017-11-29 00:00:00 +0100
tags: TestAutomation
---
<p>
Unit tests are important to test your business logic. Unit tests should not call actual services or the database and they should be able to run within a few minutes.
In this blogpost I want to give a few tips on how to make your codebase more suitable for writing unit tests.
</p>
<h3>Interfaces and dependency injection</h3>
<p>
When dealing with legacy code you will often find that different functional classes are called with the new keyword throughout other functional classes. This will often get in the way of writing a true unit test that only tests the method that you are working on.
Mitigate the problem by extracting an interface for the class whose functionality you want to access and instead pass the interface into the constructor of the class that will use the class. 
An example of code that instantiates a direct link to an Entity Framework database context is shown below.
</p>
{% highlight c# %}
public class CustomerService
{
    public IEnumerable<CustomerViewModel> GetCustomerOverview()
    {
        return (new ECommerceDbContext()).Customer
            .Where(customer => !customer.Deleted)
            .Select(MapToCustomerViewModel().Compile());
    }
	
    private Expression<Func<Customer, CustomerViewModel>> MapToCustomerViewModel()
    {
        return customer => new CustomerViewModel
        {
            Id = customer.Id,
            Name = customer.Name,
            EmailAdress = customer.EmailAdress,
            PhoneNumber = customer.PhoneNumber,
            Street = customer.Street,
            HouseNumber = customer.HouseNumber,
            HouseNumberExtension = customer.HouseNumberExtension,
            ZipCode = customer.ZipCode,
            City = customer.City,
            Country = customer.Country
        };
    }
}
{% endhighlight %}
<p>
The GetCustomerOverview method above cannot be unit tested because all calls to the method will actually instantiate the Entity Framework context with a actual calls to the underlying database.
A refactored CustomerService with interfaces passed through the constructor is shown below.
</p>
{% highlight c# %}
public class CustomerService
{
    private readonly IECommerceDbContext _eCommerceDbContext;

    public CustomerService(IECommerceDbContext eCommerceDbContext)
    {
        _eCommerceDbContext = eCommerceDbContext;
    }

    public IEnumerable<CustomerViewModel> GetCustomerOverview()
    {
        return _eCommerceDbContext.Customer
            .Where(customer => !customer.Deleted)
            .Select(MapToCustomerViewModel().Compile());
    }

    private Expression<Func<Customer, CustomerViewModel>> MapToCustomerViewModel()
    {
        return customer => new CustomerViewModel
        {
            Id = customer.Id,
            Name = customer.Name,
            EmailAdress = customer.EmailAdress,
            PhoneNumber = customer.PhoneNumber,
            Street = customer.Street,
            HouseNumber = customer.HouseNumber,
            HouseNumberExtension = customer.HouseNumberExtension,
            ZipCode = customer.ZipCode,
            City = customer.City,
            Country = customer.Country
        };
    }
}
{% endhighlight %}
<p>
The interface for my DbContext is shown below.
</p>
{% highlight c# %}
public interface IECommerceDbContext
{
    IDbSet<Customer> Customer { get; set; }

    int SaveChanges();
}
{% endhighlight %}
<p>
Now you can create a unit test to the GetCustomerOverview method. I will describe how to create a mock Entity Framework class in the next section of this post.
</p>
<p>
If you are not using Entity Framework Code First but using the database designer, you must create a separate partial class of your DbContext and set the interface inheritance in the new custom partial class. If you add an interface in the generated file, it will be lost at the next code generation from the model.
</p>
<p>
Making sure that all dependencies are passed to a class via the constructor is called dependency injection. To ensure that you do not have to new up all the dependencies in the constructors in the UI or other top layer of your application you can leverage a framework like <a href="https://msdn.microsoft.com/en-us/library/dn178463(v=pandp.30).aspx" target="_blank">Unity</a> to handle all the dependency injection for you. An example of this can be found in my <a href="https://github.com/SamanthaNeilen/ECommerceSampleApplication" target="_blank">ECommerceSampleApplication repository</a> in the ECommerceApp.NetFramework.Web project. The Unity NuGet packages combined with a unity configuration file section map all the interfaces with the concrete implementations. .Net Core has a built in dependency injection framework.
</p>
<p>
When writing tests you will want to call only specific calls of an interface during a test. You can use a mocking framework like Moq to create and verify fake behavior on interface calls. More information and examples of Moq can be found <a target="_blank" href="https://github.com/Moq/moq4/wiki/Quickstart">here</a>.
</p>
<h3>Mocking Entity Framework</h3>
<p>
When using Entity Framwork 6 and onward you can use the mock implementation in <a href="https://msdn.microsoft.com/en-us/library/dn314431(v=vs.113).aspx" target="_blank">this MSDN post</a> to simulate an entity framework without having to use a mocking framework to mock out all the calls.
</p>
<p>
An example of a testclass, using the mock framework and the refactored GetCustomerOverview method of the previous section, would look like the code snippet below (This test class uses <a href="https://github.com/nunit/docs/wiki/NUnit-Documentation" target="_blank">the NUnit framework</a> for writing and running the tests):
</p>
{% highlight c# %}
public class CustomerServiceTests
{
    private IECommerceDbContext _eCommerceDbContext;
    private CustomerService _customerService;

    [SetUp]
    public void Initialize()
    {
        _eCommerceDbContext = new MockECommerceDbContext();
        _customerService = new CustomerService(_eCommerceDbContext);

        _eCommerceDbContext.Customer.Add(new Customer { Id = 1, Name = "Test customer 1", Deleted = false });
        _eCommerceDbContext.Customer.Add(new Customer { Id = 2, Name = "Test customer 2", Deleted = true });
        _eCommerceDbContext.Customer.Add(new Customer { Id = 3, Name = "Test customer 3", Deleted = false });
    }

    [Test]
    public void GetCustomerOverview_Should_Return_Non_Deleted_Customers()
    {
        // ACT
        var result = _customerService.GetCustomerOverview();

        // ASSERT
        Assert.IsInstanceOf<IEnumerable<CustomerViewModel>>(result);            
        Assert.IsTrue(result.Any(e => e.Id == 1));
        Assert.IsFalse(result.Any(e => e.Id == 2));
        Assert.IsTrue(result.Any(e => e.Id == 3));
    }
}
{% endhighlight %}
<p>
My mock DbContext implementation is shown below:
</p>
{% highlight c# %}
public class MockECommerceDbContext : IECommerceDbContext
{
    public MockECommerceDbContext()
    {
        this.Customer = new TestDbSet<Customer>();
    }

    public IDbSet<Customer> Customer { get; set; }
    public int SaveChangesCount { get; private set; }
    public int SaveChanges()
    {
        this.SaveChangesCount++;
        return 1;
    }
}
{% endhighlight %}
<p>
The implementation of the TestDbSet is found in the MSDN post referenced earlier in this section.
When using Entity Framework below version 6 look at the same MSDN post on how to write unit tests using interfaces and mock classes for the Entity Framework database context.
</p>
<h3>Dealing with 3rd party classes</h3>
<p>
When writing .Net code you will probably use NuGet packages or other 3rd party references that do not always have interfaces available. 
The class below uses 2 external packages. FileHelpers to enable CSV exports and DotNetZip to create a ZipFile on the filesystem. 
</p>
{% highlight c# %}
public class ExportService: IExportService
{
    private readonly string _exportLocation = ConfigurationManager.AppSettings["ExportLocation"];
    private const string _customerOverviewExportName = "CustomerOverview";      

    public IZipFile CreateCustomerOverviewZipFile()
    {
        if (!Directory.Exists(_exportLocation))
        {
            Directory.CreateDirectory(_exportLocation);
        }

        var customerOverViewExportFileLocation = $"{_exportLocation}{_customerOverviewExportName}.csv";
        CreateExportFile(customerOverViewExportFileLocation);
        return ZipExportFile(customerOverViewExportFileLocation);
    }

    private ZipFile ZipExportFile(string customerOverViewExportFileLocation)
    {            
        var zipfile = new ZipFile();
        zipfile.Name = $"{_customerOverviewExportName}.zip";
        var zipentry = zipfile.AddFile(customerOverViewExportFileLocation);
        zipentry.FileName = $"{_customerOverviewExportName}.csv";
        zipfile.Save($"{_exportLocation}{zipfile.Name}");
        return zipfile;
    }

    private void CreateExportFile(string customerOverViewExportFileLocation)
    {
        var customerData = (new ECommerceDbContext()).Customer
            .Where(customer => !customer.Deleted)
            .Select(MapToCustomerExportModel().Compile());

        (new FileHelperEngine<CustomerExport>()).WriteFile(customerOverViewExportFileLocation, customerData);
    }

    private Expression<Func<Customer, CustomerExport>> MapToCustomerExportModel()
    {
        return customer => new CustomerExport
        {
            Name = customer.Name,
            EmailAdress = customer.EmailAdress,
            PhoneNumber = customer.PhoneNumber,
            Street = customer.Street,
            HouseNumber = customer.HouseNumber,
            HouseNumberExtension = customer.HouseNumberExtension,
            ZipCode = customer.ZipCode,
            City = customer.City,
            Country = customer.Country
        };
    }
}
{% endhighlight %}
<p>
The FileHelperEngine class already has an interface and can be easily refactored but the ZipFile implementation does not. The ZipFile class also does not allow itself to be mocked properly. 
</p>
<p>
A solution to mitigate these issues is to create an Inherited object on which we can create an interface and thus create a mock object. The inherited object which I have called Adapter after the <a href="https://en.wikipedia.org/wiki/Adapter_pattern" target="_blank">Adapter Design principle</a> is shown below.
</p>
{% highlight c# %}
public class ZipFileAdapter : ZipFile, IZipFile
{
    public ZipFileAdapter()
    {
        UseZip64WhenSaving = Zip64Option.AsNecessary;
        CompressionLevel = CompressionLevel.BestCompression;
    }        

    IZipEntry IZipFile.AddFile(string location)
    {
        return new ZipEntryAdapter(AddFile(location));
    }
}

public interface IZipFile
{
    Zip64Option UseZip64WhenSaving { get; set; }
    CompressionLevel CompressionLevel { get; set; }
    string Name { get; set; }
    IZipEntry AddFile(string location);
    void Save(string location);
}
{% endhighlight %}
<p>
Because I also wanted to use the created ZipEntry after calling AddFile I also had to create an Adapter for the ZipEntry class. The implementation of this class is shown below.
</p>
{% highlight c# %}
public class ZipEntryAdapter : IZipEntry
{
    private ZipEntry _zipEntry;

    public ZipEntryAdapter(ZipEntry zipEntry)
    {
        _zipEntry = zipEntry;
    }

    public string FileName { get { return _zipEntry.FileName; } set { _zipEntry.FileName = value; } }
}

public interface IZipEntry
{
    string FileName { get; set; }
}
{% endhighlight %}
<p>
Notice that in the adapters above that I only implemented and extracted the interfaces that I actually use. It is not necessary to extract all the functionality of the third party code if you never call it. 
</p>
<p>
Now that I have interfaces for all my ExportService dependencies I can refactor my ExportService class and write the appropriate unittest for the public CreateCustomerOverviewZipFile method.
</p>
{% highlight c# %}
public class ExportService
{
    private readonly string _exportLocation = ConfigurationManager.AppSettings["ExportLocation"];
    private const string _customerOverviewExportName = "CustomerOverview";
    private readonly IFileHelperEngine<CustomerExport> _fileExporter;
    private readonly IECommerceDbContext _eCommerceDbContext;

    public ExportService(IFileHelperEngine<CustomerExport> fileExporter, IECommerceDbContext eCommerceDbContext)
    {
        _fileExporter = fileExporter;
        _eCommerceDbContext = eCommerceDbContext;
    }

    public IZipFile CreateCustomerOverviewZipFile()
    {
        if (!Directory.Exists(_exportLocation))
        {
            Directory.CreateDirectory(_exportLocation);
        }

        var customerOverViewExportFileLocation = $"{_exportLocation}{_customerOverviewExportName}.csv";

        CreateExportFile(customerOverViewExportFileLocation);
        return ZipExportFile(customerOverViewExportFileLocation);
    }

    private IZipFile ZipExportFile(string customerOverViewExportFileLocation)
    {
        var container = new UnityContainer();
        container.LoadConfiguration();
        var zipfile = container.Resolve<IZipFile>();
        zipfile.Name = $"{_customerOverviewExportName}.zip";
        var zipentry = zipfile.AddFile(customerOverViewExportFileLocation);
        zipentry.FileName = $"{_customerOverviewExportName}.csv";
        zipfile.Save($"{_exportLocation}{zipfile.Name}");
        return zipfile;
    }

    private void CreateExportFile(string customerOverViewExportFileLocation)
    {
        var customerData = _eCommerceDbContext.Customer
            .Where(customer => !customer.Deleted)
            .Select(MapToCustomerExportModel().Compile());

        _fileExporter.WriteFile(customerOverViewExportFileLocation, customerData);
    }

    private Expression<Func<Customer, CustomerExport>> MapToCustomerExportModel()
    {
        return customer => new CustomerExport
        {
            Name = customer.Name,
            EmailAdress = customer.EmailAdress,
            PhoneNumber = customer.PhoneNumber,
            Street = customer.Street,
            HouseNumber = customer.HouseNumber,
            HouseNumberExtension = customer.HouseNumberExtension,
            ZipCode = customer.ZipCode,
            City = customer.City,
            Country = customer.Country
        };
    }
}

public class ExportServiceTests
{
    private readonly string _exportLocation = ConfigurationManager.AppSettings["ExportLocation"];
    private const string _customerOverviewExportName = "CustomerOverview";
    private string _customerOverViewExportFileLocation;
    private Mock<IFileHelperEngine<CustomerExport>> _mockFileExporter;
    private IECommerceDbContext _eCommerceDbContext;
    private ExportService _exportService;

    [SetUp]
    public void Initialize()
    {
        _customerOverViewExportFileLocation = $"{_exportLocation}{_customerOverviewExportName}.csv";
        _eCommerceDbContext = new MockECommerceDbContext();
        _mockFileExporter = new Mock<IFileHelperEngine<CustomerExport>>();
        _exportService = new ExportService(_mockFileExporter.Object, _eCommerceDbContext);

        _eCommerceDbContext.Customer.Add(new Customer { Id = 1, Name = "Test customer 1", Deleted = false });
        _eCommerceDbContext.Customer.Add(new Customer { Id = 2, Name = "Test customer 2", Deleted = true });
        _eCommerceDbContext.Customer.Add(new Customer { Id = 3, Name = "Test customer 3", Deleted = false });

        _mockFileExporter.Setup(exporter => exporter.WriteFile(_customerOverViewExportFileLocation, NonDeletedCustomers())).Verifiable();
    }


    [Test]
    public void CreateCustomerOverviewZipFile_Should_Export_Non_Deleted_Customers()
    {
        // ACT
        var result = _exportService.CreateCustomerOverviewZipFile();

        // ASSERT
        Assert.IsInstanceOf<IZipFile>(result);
        _mockFileExporter.Verify(exporter => exporter.WriteFile(_customerOverViewExportFileLocation, NonDeletedCustomers()));
        var mockZipfile = result as MockZipFile;
        Assert.AreEqual(1, mockZipfile.ZipEntries.Count);
        Assert.AreEqual($"{_customerOverviewExportName}.csv", mockZipfile.ZipEntries[0].FileName);
        Assert.AreEqual(_customerOverViewExportFileLocation, mockZipfile.ZipEntries[0].Location);
    }

    private IEnumerable<CustomerExport> NonDeletedCustomers()
    {
        return It.Is<IEnumerable<CustomerExport>>(customerExports =>
                        customerExports.Any(export => export.Name == "Test customer 1")
                        && customerExports.Any(export => export.Name == "Test customer 3")
                        && !customerExports.Any(export => export.Name == "Test customer 2")
                    );
    }
}
{% endhighlight %}