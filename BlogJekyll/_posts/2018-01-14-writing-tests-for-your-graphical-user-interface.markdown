---
layout: post
title:  "Writing tests for your graphical user interface"
date:   2018-01-14 00:00:00 +0100
tags: TestAutomation
---
<p>
In this blogpost I will be talking on how to create tests using Coded UI and Selenium. 
</p>
<p>
Coded UI is the UI testing framework from Microsoft for testing any type of UI that runs on the windows platform. This means you can test websites as well as WPF or other windows applications. Coded UI is shipped as a module on Visual Studio. Currently Coded UI is only shipped with Visual Studio Enterprise editions. For the currently automatable configurations see <a href="https://docs.microsoft.com/nl-nl/visualstudio/test/supported-configurations-and-platforms-for-coded-ui-tests-and-action-recordings" target="_blank">Supported configurations and platforms forCoded UI tests and action recordings</a>.
</p>
<p>
Coded UI supports IE website testing out of the box and you can install an extension to run tests in other browsers including Edge using Selenium drivers (another UI test framework). Note that the <a href="https://marketplace.visualstudio.com/items?itemName=AtinBansal.SeleniumcomponentsforCodedUICrossBrowserTesting" target="_blank">Selenium Cross Browser tools</a> have not been updated in a while and is no longer fully compatible with the latest version of the Selenium implementation. Updating the Selenium drivers of the cross-browser tools to the latest version will break functionality like finding a dropdown on a page. Therefor I recommend using Selenium to test your website if you want to test in multiple and the latest browsers that are not Internet Explorer. You can also use Selenium to automate actions within the browser windows and use Coded UI to manipulate the browser application itself.  (For example controlling the browser dialog windows.) Also Selenium is free and open source. If you do not have a Visual Studio Enterprise edition, definitely look into Selenium.
</p>
<p>
UI testing is a great method of automating regression tests for your applications. This way you can avoid human errors in repetitive tasks and when run during development of changes, you may possibly spot bugs before you hand of a new change to your QA team.
</p>
<p>
Please note that all the sample tests later in this blog post are just simple examples to help you get started. If you wish to build an extensive test library please look into building your own UI test framework using the page-object approach.
</p>
<h3>Setting up a Code UI project</h3>
<p>
To start with Coded UI make sure you have the Coded UI project type installed. It can be found under test projects.
<br/><img src="{{"/assets/images/20180114/VisualStudioTestProjects.png" | relative_url }}" alt="Visual Studio Test Project Templates"/>
</p>
<p>
If you do not see anything else than the Unit Test Project type, it means that you have not yet installed it. Open Visual Studio Installer, modify your Visual Studio Enterprise Edition and under Individual Components select the Coded UI test option under the Debugging and Testing section.
<br/><img src="{{"/assets/images/20180114/VisualStudioInstaller.png" | relative_url }}" alt="Visual Studio Installation Coded UI CheckBox"/>
</p>
<p> 
After creating your new Coded UI project, the project will add a test class and a UI Map file. Visual Studio will also startup the UIMap - Coded UI Test Builder. This tool is used to record or inspect elements on your screen. If you close it you can restart it by right clicking a .uitest file or Map file as they are called and select the Edit with Coded UI Test Builder option.
 <br/><img src="{{"/assets/images/20180114/CleanCodedUIProject.png" | relative_url }}" alt="Default Coded UI Project Files And Test Builder Window"/>
</p>
 <p>
You can now either start coding or recording tests. If you press the record button, do some clicks in any application in your windows machine and then click the generate code button. Visual Studio will generate the code for the actions you took in the CodedUITest.cs file. All code for UI controls is placed inside the UIMap.uittest file.
</p>
<p>
Recording UI tests can be helpful to start up fast but will be less maintainable in the long run, so I wish to focus on writing tests by hand for your applications. The Coded UI Test Builder is a great tool for inspecting controls on the screen and showing you the properties that Coded UI can access. These properties can also be used to search controls as I will explain later on.
</p>
<p>
To use the Coded UI Test Builder element inspector, click and drag the circle button to any UI element that you want to inspect. You will see a blue highlight on the control that is focused. When releasing your mouse, a properties window will popup showing you the details of the control that you have selected. When inspecting an UI element in a browser it may take a second or so for the highlight to get into the page. 
</p>
<p>
A windows application example of inspecting a button is shown below:
<br/><img src="{{"/assets/images/20180114/WindowsButtonProperties.png" | relative_url }}" alt="Windows Button Properties"/>
 </p>
 <p>
And here is an example of inspecting a button in a web page:
<br/><img src="{{"/assets/images/20180114/WebButtonProperties.png" | relative_url }}" alt="Web Button Properties"/>

 </p>
  <h3>Setting up a simple WPF test with Coded UI</h3>
 <p>
 A working sample of the code below including the sample application that it is run on can be found in my <a href="https://github.com/SamanthaNeilen/CodedUITestApplication" target="_blank">CodedUITestApplication repository</a>.
 </p>
 <p>
The class containing the test must be decorated with the [CodedUITest] attribute. The test method itself is decorated with a [TestMethod] attribute.
</p>
{% highlight c# %}
using Microsoft.VisualStudio.TestTools.UITesting;
using Microsoft.VisualStudio.TestTools.UITesting.WinControls;
using Microsoft.VisualStudio.TestTools.UITesting.WpfControls;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace CodedUITestTestApplication.Tests
{
    [CodedUITest]
    public class WindowsAppTests
    {
        [TestMethod]
        public void Test_WPF_Addition()
        {
            var app = ApplicationUnderTest.Launch(@"..\..\..\CodedUITestApplication.WPF\bin\Debug\CodedUITestApplication.WPF.exe");

            var amount1Edit = new WpfEdit(app);
            amount1Edit.SearchProperties.Add(WpfEdit.PropertyNames.AutomationId, "txtFirstAmount");
            amount1Edit.Text = "5";

            var amount2Edit = new WpfEdit(app);
            amount2Edit.SearchProperties.Add(WpfEdit.PropertyNames.AutomationId, "txtSecondAmount");
            amount2Edit.Text = "10";

            var buttonPlus = new WpfButton(app);
            buttonPlus.SearchProperties.Add(WpfEdit.PropertyNames.AutomationId, "btnAdd");
            Mouse.Click(buttonPlus);

            var resultLabel = new WpfText(app);
            resultLabel.SearchProperties.Add(WpfEdit.PropertyNames.AutomationId, "lblResult");
            Assert.AreEqual("15", resultLabel.DisplayText);
        }
    }
}
{% endhighlight %}
<p>
As you can see, you find WPF controls by adding search properties. As long as you are adding search properties, your object in memory will not actually search for the control until you either call TryFind, use the control as parent for another new control, change a property or try to invoke an action on the control in the application. If a control is not found when calling TryFind, false is returned. In all other cases a ControlNotFound exception is thrown.
</p>
<p>
The framework uses certain timeouts when finding controls. These timeouts can be changed by accessing the static PlayBack class.
</p>
<h3>Setting up a simple Windows application test with Coded UI</h3>
<p>
In this example I just boot up and automate the windows calculator.
</p>

{% highlight c# %}
using Microsoft.VisualStudio.TestTools.UITesting;
using Microsoft.VisualStudio.TestTools.UITesting.WinControls;
using Microsoft.VisualStudio.TestTools.UITesting.WpfControls;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace CodedUITestTestApplication.Tests
{
    [CodedUITest]
    public class WindowsAppTests
    {
        [TestMethod]
        public void Test_Windows_Calculator()
        {
            var app = ApplicationUnderTest.Launch(@"C:\Windows\system32\calc.exe");

            var button5 = new WinButton(app);
            button5.SearchProperties.Add(WinControl.PropertyNames.Name, "5");
            Mouse.Click(button5);

            var buttonPlus = new WinButton(app);
            buttonPlus.SearchProperties.Add(WinControl.PropertyNames.Name, "Add");
            Mouse.Click(buttonPlus);

            Mouse.Click(button5);

            var buttonEquals = new WinButton(app);
            buttonEquals.SearchProperties.Add(WinControl.PropertyNames.Name, "Equals");
            Mouse.Click(buttonEquals);

            var resultText = new WinText(app);
            resultText.SearchProperties.Add(WinControl.PropertyNames.Name, "Result");
            Assert.AreEqual("10", resultText.DisplayText);
        }
    }
}
{% endhighlight %}
<p>
Notice that the searching and asserting is the same as with the WPF application. Only the base class for defining the type of control is different. 
</p>
<h3>Setting up a simple web test with Coded UI</h3>
<p>
A webtest is quite similar. Here again the base class for the control is different. Also, we start a BrowserWindow instead of using ApllicationUnderTest. BrowserWindow is actually inherited from ApplicationUnderTest. You can use BrowserWindow.CurrentBrowser (before calling Launch) to determine the browser to use. The default is "IE" and you can use "Chrome" or "Firefox" to start other browsers. Note that Firefox will not start if your browser version is not supported by the  <a href="https://marketplace.visualstudio.com/items?itemName=AtinBansal.SeleniumcomponentsforCodedUICrossBrowserTesting" target="_blank">Selenium Cross Browser tools (version > 47.0.1, 32 bit)</a>.
</p>
{% highlight c# %}
using CodedUITestApplication.Shared.Resources;
using Microsoft.VisualStudio.TestTools.UITesting;
using Microsoft.VisualStudio.TestTools.UITesting.HtmlControls;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;
using System.Configuration;
using System.Linq;

[CodedUITest]
public class WebTests
{
    protected string StartUrl = ConfigurationManager.AppSettings["ApplicationStartUrl"];

    [TestMethod]
    public void WebTest_TestForm()
    {           
        BrowserWindow browser = BrowserWindow.Launch(new Uri(StartUrl));

        var nameTextBox = new HtmlEdit(browser);
        nameTextBox.SearchProperties.Add(HtmlControl.PropertyNames.Id, "Name");
        nameTextBox.Text = "Some Name";

        var submitButton = new HtmlButton(browser);
        submitButton.SearchProperties.Add(HtmlButton.PropertyNames.Type, "submit");
        Mouse.Click(submitButton);

        var header = new HtmlControl(browser);
        header.SearchProperties.Add(HtmlControl.PropertyNames.Id, "addCustomerHeader");
        Assert.IsTrue(header.TryFind());

        var nameValidationmessage = new HtmlSpan(browser);
        nameValidationmessage.SearchProperties.Add(HtmlControl.PropertyNames.Id, "Name-error");
        Assert.IsFalse(nameValidationmessage.TryFind());

        var expectedEmailvalidationMessage = string.Format(Messages.FieldRequired, Labels.EmailAddress);
        var emailAddessValidationMessage = new HtmlSpan(browser);
        emailAddessValidationMessage.SearchProperties.Add(HtmlControl.PropertyNames.Class, "field-validation-error");
        emailAddessValidationMessage.SearchProperties.Add(HtmlControl.PropertyNames.InnerText, expectedEmailvalidationMessage, 
		    PropertyExpressionOperator.Contains);

        Assert.IsTrue(emailAddessValidationMessage.TryFind());
        Assert.IsTrue(emailAddessValidationMessage.InnerText.Equals(expectedEmailvalidationMessage));
    }
}
{% endhighlight %}
<h3>Running a single test on multiple browsers using a data source file</h3>
<p>
<b>[Edit Feb 16 2018]</b> Note that the solution described below works for projects using the Coded UI framework. When using the default Microsoft Test V2 framework, the use of the TestSettings files is deprecated and your tests will stay undiscoverable in the Test Explorer window. See the <a href="https://github.com/SamanthaNeilen/WebDriverTestApplication" target="">WebDriverTestApplication repository</a> for running tests with the MSTest V2 framework. <b>[End edit Feb 16 2018]</b>
</p>
<p>
The default unit test framework from Microsoft can consume input files to run your tests for multiple data rows. In the following example I have added a CSV with the browsers names I want to test as data source. When this test is run the test will execute for each browser in sequence.
</p>
{% highlight c# %}
using CodedUITestApplication.Shared.Resources;
using Microsoft.VisualStudio.TestTools.UITesting;
using Microsoft.VisualStudio.TestTools.UITesting.HtmlControls;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;
using System.Configuration;
using System.Linq;

[CodedUITest]
public class WebTests
{
    private TestContext _testContextInstance;
    public TestContext TestContext
    {
        get { return _testContextInstance; }
        set { _testContextInstance = value; }
    }

    protected string StartUrl = ConfigurationManager.AppSettings["ApplicationStartUrl"];

    [TestMethod, DataSource("Browsers")]
    public void WebTest_TestForm()
    {
        var browserIdentifier = "IE";
        if (TestContext.DataRow != null)
        {
            browserIdentifier = TestContext.DataRow[0].ToString();
        }
        BrowserWindow.CurrentBrowser = browserIdentifier;
        BrowserWindow browser = BrowserWindow.Launch(new Uri(StartUrl));

        var nameTextBox = new HtmlEdit(browser);
        nameTextBox.SearchProperties.Add(HtmlControl.PropertyNames.Id, "Name");
        nameTextBox.Text = "Some Name";

        var submitButton = new HtmlButton(browser);
        submitButton.SearchProperties.Add(HtmlButton.PropertyNames.Type, "submit");
        Mouse.Click(submitButton);

        var header = new HtmlControl(browser);
        header.SearchProperties.Add(HtmlControl.PropertyNames.Id, "addCustomerHeader");
        Assert.IsTrue(header.TryFind());

        var nameValidationmessage = new HtmlSpan(browser);
        nameValidationmessage.SearchProperties.Add(HtmlControl.PropertyNames.Id, "Name-error");
        Assert.IsFalse(nameValidationmessage.TryFind());

        var expectedEmailvalidationMessage = string.Format(Messages.FieldRequired, Labels.EmailAddress);
        var emailAddessValidationMessage = new HtmlSpan(browser);
        emailAddessValidationMessage.SearchProperties.Add(HtmlControl.PropertyNames.Class, "field-validation-error");
        emailAddessValidationMessage.SearchProperties.Add(HtmlControl.PropertyNames.InnerText, expectedEmailvalidationMessage, 
		     PropertyExpressionOperator.Contains);

        Assert.IsTrue(emailAddessValidationMessage.TryFind());
        Assert.IsTrue(emailAddessValidationMessage.InnerText.Equals(expectedEmailvalidationMessage));
    }
}
{% endhighlight %}
<p>
To ensure that the Browsers datasource is present in your project, first add the file you want to use as a data source to the project. Set the properties of the file to Content and Copy always.
<br/><img src="{{"/assets/images/20180114/FileBuildAction.png" | relative_url }}" alt="File Properties Build Action"/>
</p>
<p>
Next define the data source connection string in the App.config of the test project. You will also need to add a configuration section to ensure that the configuration elements will be processed. 
{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <configSections>
    <section name="microsoft.visualstudio.testtools" type="Microsoft.VisualStudio.TestTools.UnitTesting.TestConfigurationSection, 
	    Microsoft.VisualStudio.QualityTools.UnitTestFramework, Version=10.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"/>
  </configSections>  
  <connectionStrings>
    <add name="Browsers" connectionString="Browsers.csv" providerName="Microsoft.VisualStudio.TestTools.DataSource.CSV" />
  </connectionStrings>
  <microsoft.visualstudio.testtools>
    <dataSources>
      <add name="Browsers" connectionString="Browsers" dataTableName="Browsers#csv" dataAccessMethod="Sequential"/>
    </dataSources>
  </microsoft.visualstudio.testtools>
</configuration>
{% endhighlight %}
</p>
<p>
Next add a TestSettings file to the solution.
<br/><img src="{{"/assets/images/20180114/VisualStudioTestSettings.png" | relative_url }}" alt="Visual Studio Test Settings File Type"/>
</p>
<p>
Open the test settings file and navigate to deployment. Check the Enable Deployment option and add the csv file that needs to be deployed when running the test. 
<br/><img src="{{"/assets/images/20180114/DeploymentSettingBrowsers.png" | relative_url }}" alt="Deployment settings Browser tag"/>
</p>
<p>
Sometimes Visual Studio will not automatically pick up the TestSettings file. If you get errors about not finding the data source first check if the TestSettings file is selected for use.
<br/><img src="{{"/assets/images/20180114/VisualStudioTestSettingsMenu.png" | relative_url }}" alt="Visua Studio Test Settings Menu Option"/>
</p>
<h3>Setting up a simple web test with Selenium</h3>
<p>
<b>[Edit Feb 16 2018]</b> Note that the solution described below works for projects using the Coded UI framework. When using the default Microsoft Test V2 framework, the use of the TestSettings files is deprecated and your tests will stay undiscoverable in the Test Explorer window. See the <a href="https://github.com/SamanthaNeilen/WebDriverTestApplication" target="">WebDriverTestApplication repository</a> for running tests with the MSTest V2 framework. <b>[End edit Feb 16 2018]</b>
</p>
<p>
In this last section I wanted to show a web test using Selenium. To be able to use Selenium in your test project first add the Selenium.Support NuGet package. This package will also automatically add the Selenium.Webdriver package.
<br/><img src="{{"/assets/images/20180114/SeleniumNuGetPackages.png" | relative_url }}" alt="Selenium NuGet Packages"/>
</p>
<p>
Next you need to download the driver executables that will actually start and control the browser. The latest driver executables can be found on the <a href="http://www.seleniumhq.org/download/" target="_blank">Selenium download page</a>.
Add them to your project and make sure that they get deployed to either the bin folder of the project or add them to the deployment configuration in the TestSettings file as explained in the "Running a single test on multiple browsers using a data source file" section of this post, so that they are in the same directory as the test project output dll.
<br/><img src="{{"/assets/images/20180114/SeleniumDrivers.png" | relative_url }}" alt="Selenium Driver Files"/>
<br/><img src="{{"/assets/images/20180114/DeploymentSettingsSelenium.png" | relative_url }}" alt="Deployment Settings Selenium"/>
</p>
<p>
After setting up, you can use the test method below to check the same functionality as the Coded UI web test in the last section. Note the different syntax but the same general idea of searching and manipulating controls.
</p>
{% highlight c# %}
using CodedUITestApplication.Shared.Resources;
using Microsoft.VisualStudio.TestTools.UITesting;
using Microsoft.VisualStudio.TestTools.UITesting.HtmlControls;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.IE;
using OpenQA.Selenium.Firefox;
using System;
using System.Configuration;
using System.Linq;

namespace CodedUITestTestApplication.Tests
{
    [CodedUITest]
    public class WebTests
    {
        [TestMethod, DataSource("Browsers")]
        public void WebTest_Selenium_TestForm()
        {
            var browserIdentifier = "Chrome";
            if (TestContext.DataRow != null)
            {
                browserIdentifier = TestContext.DataRow[0].ToString();
            }

            using (var driver = StartWebdriver(browserIdentifier))
            {
                driver.Navigate().GoToUrl(new Uri(StartUrl));

                var nameTextBox = driver.FindElement(By.Id("Name"));
                nameTextBox.SendKeys("Some Name");

                var submitButton = driver.FindElement(By.TagName("button"));
                submitButton.Click();

                var header = driver.FindElement(By.Id("addCustomerHeader"));
                Assert.IsTrue(header.Displayed);

                Assert.AreEqual(0, driver.FindElements(By.Id("Name-error")).Count);

                var expectedEmailvalidationMessage = string.Format(Messages.FieldRequired, Labels.EmailAddress);
                var validationMessages = driver.FindElements(By.CssSelector(".field-validation-error"));
                Assert.IsTrue(validationMessages.Any(element => element.Displayed 
                    && ChildSpanContainsExpectedText(element, expectedEmailvalidationMessage)));

                driver.Close();
            }
        }

        private static bool ChildSpanContainsExpectedText(IWebElement element, string expectedEmailvalidationMessage)
        {
            return element.FindElement(By.TagName("span")).Text.Equals(expectedEmailvalidationMessage);
        }

        private IWebDriver StartWebdriver(string browserIdentifier)
        {
            switch (browserIdentifier)
            {
                case "IE":
                    return new InternetExplorerDriver();
                case "Firefox":
                    return new FirefoxDriver();
                case "Chrome":
                    return new ChromeDriver();
                default:
                    throw new NotSupportedException();
            }
        }
    }
}
{% endhighlight %}

<h3>Resources</h3>
<p>
<a href="https://app.pluralsight.com/library/courses/codedui-test-automation/table-of-contents" target="_blank">Pluralsight- Coded UI</a><br/>
<a href="https://app.pluralsight.com/library/courses/selenium/table-of-contents" target="_blank">Pluralsight- Selenium</a><br/>
<a href="https://docs.microsoft.com/nl-nl/visualstudio/test/use-ui-automation-to-test-your-code" target="_blank">Microsoft docs - Use UI Automation To Test Your Code</a><br/>
<a href="http://www.seleniumhq.org/" target="_blank">Selenium HQ</a><br/>
</p>