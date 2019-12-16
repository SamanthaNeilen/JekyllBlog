---
layout: post
title:  "Managing a SQL Server database from a Visual Studio Database project"
date:   2017-11-24 00:00:00 +0100
tags: VisualStudio SQLServer
---
<p>
	When building an application I also want to manage changes to the database from source control. One of the ways of doing this is with a Visual Studio SQL Server Database project. It integrates well with source control and can be integrated into automatic deployments.
</p>
<p>
	All screenshots and used techniques that are written in this post are available within Visual Studio 2017 community edition and SQL Server 2017 express edition that are both free downloads.
</p>
<p>
	The SQL Server Database project type can be found in the New Project window under de SQL Server templates as shown below:<br/>
	<img src="{{"/assets/images/20171124/DatabaseProject.png" | relative_url }}" alt="Database Project"/>
<p>
<p>
	This project will just create an empty project without any objects.<br/>
	<img src="{{"/assets/images/20171124/EmptyDatabaseProjectSolution.png" | relative_url }}" alt="Empty Database Project Solution Explorer"/>
</p>
<p>
	You can now start adding new items like tables, views, stored procedures and all kinds of other SQL Server database objects. A good practice is to put these objects into folders representing the different database objects for better maintainability.
	You can also import objects from an existing database by right-clicking on the project and select Import from the available options ( the import database and dacpac options are only available when the database project is still empty).
</p> 
<p>
	When creating an object like a table, Visual Studio will show an editor as shown in the image below. You can leverage the designer combined with the properties window to define fields and column settings or just type in the SQL statements in the T-SQL window on the lower left side.
	Below you can see an example of how I created a simple Customer table.<br/>
	<img src="{{"/assets/images/20171124/DesignView.png" | relative_url }}" alt="Designer View Visual Studio"/>
</p>
<p>
	After creating or changing objects in your project you can publish these changes to a database with the Schema Compare option in the project context menu.<br/>
	<img src="{{"/assets/images/20171124/SchemaCompareContextMenu.png" | relative_url }}" alt="Schema Compare Context Option"/>
</p>
<p>
	Use the select target drop down to go through the dialogs to select the database that you want to publish to.<br/>
	<img src="{{"/assets/images/20171124/SchemaCompareSelectTarget.png" | relative_url }}" alt="Schema Compare Select Target"/>
</p>
<p>
	After selecting the database the Compare button on the upper left will become enabled and you can click it to kick off the compare between your database and the project. You can click on an object to see the changes between the specific objects.<br/>
	<img src="{{"/assets/images/20171124/SchemaCompareTableChanges.png" | relative_url }}" alt="Schema Compare Table changes"/>
</p>
<p>
	If you want to update the database to reflect the changes made, click the update button. You can deselect unwanted changes with the checkboxes in the Action column of the SchemaCompare tab. Visual Studio will show if the changes were successful and give you some options to view details.<br/>
	<img src="{{"/assets/images/20171124/SchemaCompareUpdateSuccesful.png" | relative_url }}" alt="Schema Compare Update Succesful"/>
</p>
<p>
	When the update fails, for example when you add a new non-nullable column without a default value or deleting a column with data, the results will show like below:<br/>
	<img src="{{"/assets/images/20171124/SchemaCompareUpdateFailed.png" | relative_url }}" alt="Schema Compare Update Failed"/>
</p>
<p>
	By clicking View Results you see the script that was run but also a tab that shows the message of what went wrong.<br/>
	<img src="{{"/assets/images/20171124/SchemaCompareUpdateFailedMessage.png" | relative_url }}" alt="Schema Compare Update Failed Message"/>
</p>
<p>
	You can either force the data loss by changing the schema compare options via the gear icon of the schema compare tab  as shown below or run an update script to mitigate the data loss.<br/>
	<img src="{{"/assets/images/20171124/SchemaCompareOptions.png" | relative_url }}" alt="Schema Compare Options"/>
</p>
<p>
	In my case, I had a faulty existing Countr column that did not match a newly created Country column. To get Schema Compare to update the database I had to create a data migration pre-upgrade script as shown below.
</p>
{% highlight sql %}
IF EXISTS(SELECT 1 FROM sys.columns WHERE Name = N'Countr' AND Object_ID = Object_ID(N'dbo.Customer'))
	AND NOT EXISTS(SELECT 1 FROM sys.columns WHERE Name = N'Country' AND Object_ID = Object_ID(N'dbo.Customer'))
BEGIN
	ALTER TABLE Customer ADD Country NVARCHAR(100) NULL
 
	DECLARE @sqlCommand varchar(1000) = 'UPDATE Customer SET Country = Countr'
	EXEC(@sqlCommand)
 
	ALTER TABLE Customer DROP Countr
END
{% endhighlight %}
<p>
	The update is a dynamic statement  because the script will not compile with a column name that does not exist before the upgrade script has been run. Notice that the IF statement is very specific to this change in the database. This is because we can add this script as a Pre-Deployment Script to the database project.<br/>
	<img src="{{"/assets/images/20171124/ScriptFileOptions.png" | relative_url }}" alt="Script File Options"/>
</p>
<p>
	Even if we have to run the script manually when using a schema compare, Pre- and Post-Deployment scripts will be run automatically when running the database project (F5) on a database, or when deploying the bacpac file that is the output of this project (found in the bin folder after building the project). The connection string that is used when running/debugging the project can be found and changed in the properties pages of the project in the Debug tab. There is also a SQL Script in the output folder that can be used to upgrade a database. The Pre- and Post-Deployment scripts are contained within the SQL script that is the output of the project.<br/>
	<img src="{{"/assets/images/20171124/ProjectPropertyPages.png" | relative_url }}" alt="Project Property Pages"/>
</p>
<p>
	Once all known instances of the database are updated with the specific Pre-Deployment script. It can be removed to avoid any clutter of the project. 
	The specific IF statement will ensure that the statements are only run on a database that can do the specific data migration and skips it when creating a completely new instance.
</p>
<p>
	These projects are really only for managing schema information, not data. If you want to secure your data, you need to create regular (scheduled) backups of your database. 
	We can use Post-Deployment scripts to add seed data to tables when they are empty. An example is shown below:
</p>
{% highlight sql %}
IF NOT EXISTS(SELECT TOP(1) [Name] FROM Customer)
BEGIN
	INSERT INTO Customer ([Name],[Emailadress],[Phonenumber],[Street], [Housenumber], [HousenumberExtension], [Zipcode], [City], [Country])
	VALUES ('My test customer 1', 'company@testcompany1.com', '0123456789', 'TestStreet', 4, NULL, '1111AA', 'TestCity', 'TestCountry');
	INSERT INTO Customer ([Name],[Emailadress],[Phonenumber],[Street], [Housenumber], [HousenumberExtension], [Zipcode], [City], [Country])
	VALUES ('My test customer 2', 'info@somecompany.com', '0112233445', 'SomeStreet', 2, 'a', '5555UD', 'SomeCity', 'TestCountry');
	INSERT INTO Customer ([Name],[Emailadress],[Phonenumber],[Street], [Housenumber], [HousenumberExtension], [Zipcode], [City], [Country])
	VALUES ('My test customer 3', 'company3@test.com', '0997654432', 'StreetStuff', 4, NULL, '8261SK', 'CityStuff', 'TestCountry');
END
{% endhighlight %}
<p>
	So now we have project that can be plugged into source control. All the objects are stored as SQL files and work well in source control comparing and merging.
	If you have any manual scripts, that you might want to run periodicly, store them with the solution in a seperate solution items folder so they will not be compiled into the project output.
</p>
<p>
	The dacpac files that are the output of the project can be used by an automated or manual deployment process to change the databases in your environments. Once you update the database, it will remember the version of the dacpac used to update the database. The project has a refactor log file reflecting all changes made in a sequential form. Once  a dacpac version has been applied with the changes and you want to run the dacpac again after making manual modifications to the database, it will not be able to restore properly to the intended database state. This is why during development you will usually be using the Schema Compare functionality and running 
	Pre- and Post-Deployment scripts manually until all changes are ready and the dacpac reflecting the next sequential update will be pushed out to your testing and eventually production environments.
</p>
<p>
	If you have a dacpac file you can deploy it in manually in SQL Server Management Studio with the option below for a new database:<br/>
	<img src="{{"/assets/images/20171124/ManagementStudioDeploy.png" | relative_url }}" alt="SQL Management Studio Deploy"/>
</p>
<p>
	Or use the option below on an existing database to upgrade it:<br/>
	<img src="{{"/assets/images/20171124/ManagementStudioUpgrade.png" | relative_url }}" alt="SQL Management Studio Upgrade"/>
</p>
<p>
	If you are using VSTS for automatic deployments, you can add a step to deploy dacpac files.
</p>
<p>
	Below are some references to documentation about the topics discussed in this post:<br/>
	<a href="https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications" target="_blank">Microsoft Docs page for data-tier applications</a> <br/>
	<a href="https://docs.microsoft.com/en-us/visualstudio/data-tools/creating-and-managing-databases-and-data-tier-applications-in-visual-studio" target="_blank">Microsoft Docs page for database projects</a>
</p>
