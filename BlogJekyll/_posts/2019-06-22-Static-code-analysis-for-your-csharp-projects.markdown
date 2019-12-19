---
layout: post
title:  "Static code analysis for your C# projects"
date:   2019-06-22 00:00:00 +0100

tags: VisualStudio StaticCodeAnalysis
---

Static code analysis analyzes your source code or compiled DLL files for certain patterns or filenames. There are several code analyzers available for C# in Visual Studio and/or Azure DevOps. These code analyzers improve consistency, prevent technical debt and prevent security issues.

The older Code Analysis features in Visual Studio (Analyze > Run Code Analysis option) and Project properties (Code Analysis tab) are marked as deprecated and will not be covered in this post.

**Table of contents:**
* Table of Contents
{:toc}

### Visual Studio built-in analyzers

The Visual Studio compiler (Roslyn) already has some built-in analyzer rules. When loading a project (like the [eShopOnWeb](https://github.com/dotnet-architecture/eShopOnWeb) reference implementation) and opening a file, the analyzers installed in Visual Studio will analyze the file and show any error, warning or information message in the Visual Studio error list window. 

![[Default style messages]]({{"/assets/images/20190622/Default-style-messages.png" | relative_url }})

By default, Visual Studio will only contain some analyzers for code styling that output as information messages. You can edit the default code analyzer rules in the Visual Studio options via:

Tools > Options > Text Editor > C# > Code Style 

![[Default style messages settings]]({{"/assets/images/20190622/Default-style-messages-settings.png" | relative_url }})

The severity of the ruleset can be changed in this options window:

![[Default style messages settings set severity]]({{"/assets/images/20190622/Default-style-messages-set-to-warning.png" | relative_url }})

By default, it only analyzes the open file but you can enable solution-wide analysis in the Visual Studio options to asynchronously analyze the entire solution. This option is found via:

Tools > Options > Text Editor > C# > Advanced > Enable full solution analysis 

![[Enable full solution analysis]]({{"/assets/images/20190622/Enable-full-solution-analysis.png" | relative_url }})

If you use Visual Studio 2019 and you have set rules resulting in errors or warnings, you will also see a counter at the bottom of the file alerting you to issues in the files. There are also the colored squiggly lines in the text editor below the statement that may have and issue. There are colored blocks in the scrollbar of the text editor indicating where the issues are located in a file. And if you have the [Productivity Power Tools](https://marketplace.visualstudio.com/items?itemName=VisualStudioPlatformTeam.ProductivityPowerPack2017) extension installed the Solution Error Visualization will show the same colored squiggly lines in the Solution Explorer window below filenames that have issues.

I have marked all visual indicators for errors and or warnings in the screenshot below:

![[Visual indicators for warnings or errors]]({{"/assets/images/20190622/Visual-indicators-for-issues.png" | relative_url }})

### SonarLint

For extra code analysis, I use the [SonarLint extension](https://www.sonarlint.org/visualstudio/) in Visual Studio. This extension loads extra code analysis rules for several categories (code smells, bugs and security issues). For the full list see the [SonarSource rules pages](https://rules.sonarsource.com/csharp). Rules from SonarLint can be identified by the S prefix in the Error list window.

![[SonarLint warnings example]]({{"/assets/images/20190622/SonarLint-messages.png" | relative_url }})

You can use a .ruleset file in your project to disable or change the default severities or disable warnings for the default rules. See the [Microsoft Docs page for adding ruleset files](https://docs.microsoft.com/en-us/visualstudio/code-quality/how-to-create-a-custom-rule-set) for more instructions.

When using SonarLint, I usually enable the category column in my error list window to triage the warnings shown. You can right-click the error list window and use the option Show Columns > Category to add it.

![[Add Category column to error list]]({{"/assets/images/20190622/Add-Category-column-to-error-list.png" | relative_url }})

### SonarQube

The SonarLint extension also enables integration with a [SonarQube server](https://www.sonarqube.org/). SonarQube is open source static code analysis platform that can integrate [with Visual Studio](https://www.sonarlint.org/visualstudio/#visualstudio-connected-mode) and [with Azure DevOps](https://www.azuredevopslabs.com/labs/vstsextend/sonarqube/). SonarQube can be used to define a ruleset that all team members can download into new or existing projects. SonarQube (when integrating with Azure DevOps) can also provide code coverage metrics and code duplication analysis. It can also provide insight into the number of issues over time and provides a technical debt score for a solution. You can also fail a build if your solution does not meet a configured quality gate. SonarQube is great in providing code analysis and related dashboards especially when working with a team on a code project. See the [SonarQube](https://www.sonarqube.org/) website for all the features and installation instructions.

### Security Code Scan

[Security Code Scan](https://security-code-scan.github.io/) is a static analyzer extension focusing on security issues in your code. It checks for patterns that indicate SQL injection or XSS vulnerabilities in your code and several other issues that are defined by [OWASP](https://www.owasp.org/index.php/Main_Page) as security issues.

After installing the extension (and enabling full solution-wide analysis) the warnings from Security Code Scan are listed with the prefix SCS.

![[Security Code Scan messages]]({{"/assets/images/20190622/Security-Code-Scan-messages.png" | relative_url }})

### Audit.NET

[Audit.NET](https://github.com/OSSIndex/audit.NET) is an [extension for Visual Studio](https://marketplace.visualstudio.com/items?itemName=VorSecurity.AuditNet) that scans your package.config file and compares the package references against several public databases containing known vulnerabilities. Any issues with packages are shown in the Error window as errors. These errors are not blocking and will not result prevent you from building, debugging or running your solutions.

![[Audit.NET messages]]({{"/assets/images/20190622/AuditNET-errors.png" | relative_url }})

At the time of this writing, the extension can analyze .NET Core projects (and the new package references) but it does not seem to properly show the current issues in the error window.

### WhiteSource Bolt

[WhiteSoure Bolt](https://bolt.whitesourcesoftware.com/azure/) is the free version of WhiteSource and can be integrated [with Azure DevOps](https://www.azuredevopslabs.com/labs/vstsextend/whitesource/). The free version can be used in a commercial environment but beware of the [terms of service](https://bolt.whitesourcesoftware.com/azure/terms/) data and usage policies as it does send and store metadata and file hashes to WhiteSource hosted in America. This may be an issue in certain corporate environments (especially in Europe).

WhiteSource Bolt analyzes your project and will report on NuGet packages or included DLL files with known vulnerabilities. It will also give you an overview of all used 3rd party components and their licenses.

See [the Azure DevOps labs last step of the Trigger a build section](https://www.azuredevopslabs.com/labs/vstsextend/whitesource/) for screenshots of the report for a build.

