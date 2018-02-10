---
layout: post
title:  "Setting up a build using VSTS"
date:   2018-02-10 00:00:00 +0100
tags: DevOps
---
<p>
VSTS is the online platform from Microsoft to save your code. A VSTS project is free when there are 5 users or less and can function as a free private source control for small teams and projects. It also has Scrum/Kanban board functionality to manage a backlog and issues. But the best feature might be that you can use the  build and deployment features to fully adopt a DevOps way of working.  In this blogpost I will be showing how to create and run a build using VSTS. In a next blog post I will cover how to set up a deployment to Azure using the output of the build.
</p>
<p>
Even if you are not using a build for creating deployments, they still ensure that the source repository does not only build on your machine. It can also run your unit tests for you if you forget to run them yourself. 
</p>
<p>
In the past you always needed to set up a server or workstation with a build agent installed to set up builds, but I recently learned that there are Hosted Agents that actually spin up a build agent for you in the cloud and publish the build output (deployment assets) to an output folder. You can then use the deployment pipelines to deploy these assets to Azure or a server. To see what builds are supported by the Hosted Agent see the <a href="https://docs.microsoft.com/en-us/vsts/build-release/concepts/agents/hosted" target="_blank">Microsoft Docs page</a>. 
<p>
To set up a VSTS project visit <a href="https://www.visualstudio.com/team-services" target="_blank">https://www.visualstudio.com/team-services/</a>. You can create builds for other source control providers like GitHub so you can create a build but will not have to migrate your source control and lose any check-in history.
</p>
<p>
<h3>Setting up a build</h3>
<p>
In this example I will be setting up a build for my <a href="https://github.com/SamanthaNeilen/ECommerceSampleApplication" target="blank">ECommerceSampleApplication repository</a> that is hosted on GitHub. It contains a .Net Framework 4.6.1 MVC application, a supporting database project and some unit tests.
</p>
<p>
First go to your team project. Go to the builds section under Build and Release. Then click the New button to start a new build definition.
<br/><img src="{{"/assets/images/20180210/CreateNewBuildDefinition.png" | relative_url }}" alt="Create New Build Definition"/>
</p> 
<p>
I want to create a build for my GitHub repository so I choose GitHub as my provided and created a connection to GitHub using my username and password. (Note that the process of creating the connection to GitHub uses several pop-ups and may not work properly in all browsers.)
<br/><img src="{{"/assets/images/20180210/ConnectToSource.png" | relative_url }}" alt="Connect To Source Control Provider"/>
</p>
<p>
After clicking continue you get the option of selecting a specific build template for your type of project. If you want more details on the templates or the build steps included, see <a href="https://docs.microsoft.com/en-us/vsts/build-release/" target="_blank">https://docs.microsoft.com/en-us/vsts/build-release/</a>.
<br/><img src="{{"/assets/images/20180210/SelectBuildTemplate.png" | relative_url }}" alt="Select A Build Template"/>
</p>
<p>
I chose the ASP.Net project template. It gets my sources, restores my NuGet packages, builds the solution, runs the unittests, sets up publishing of the pdb files for remote debugging and publishes the build artifacts (project output) to the VSTS deployment/download location. 
<br/><img src="{{"/assets/images/20180210/BuildSolutionStepDefaults.png" | relative_url }}" alt="Build Solution Step Defaults"/>
</p>
<p>
I can actually just use most of the defaults for my project. I don’t need the publish symbol paths task so I deleted it. In the Build Solution Task, note the input field for Platform and Configuration. These variables will be populated with values from the Variables tab. In this task also notice the MSBuild arguments. This configuration will output the deployment package for my web package to the artifact staging directory. Seeing as my MVC project is not only the project I wanted to build and deploy, I removed everything in the MSBuild Arguments except the /p DeployOnBuild=true argument. 
<br/><img src="{{"/assets/images/20180210/BuildSolutionStepAfterChanges.png" | relative_url }}" alt="Build Solution Step After Changes"/>
</p> 
<p>
Next I added 2 custom steps to copy the deployment files from the website and database projects. The Copy tasks are shown in the images below.  (All advanced and control options settings were not changed from the defaults)
<br/><img src="{{"/assets/images/20180210/CopyWebFilesTask.png" | relative_url }}" alt="Copy Web Files Task"/>
<br/><img src="{{"/assets/images/20180210/CopyDatabaseFilesTask.png" | relative_url }}" alt="Copy Database Files Task"/>
</p>
<p>
If you have tests in your solution that run as integration tests and cannot and connect to resources like the database or a webservice from the test run process, you could choose exclude them in the Test Assemblies task via the Test filter criteria box. 
<br/><img src="{{"/assets/images/20180210/TestFilterAttribute.png" | relative_url }}" alt="Test Filter Attribute Field"/>
</p>
<p> 
Most input fields in the tasks have an information icon to figure out how the fields are used. Also again look at <a href="https://docs.microsoft.com/en-us/vsts/build-release/" target="_blank">https://docs.microsoft.com/en-us/vsts/build-release/</a> for more guidance on specific needs.
</p>
<p>
Also check out the settings under the other tabs of the build. 
<br/><img src="{{"/assets/images/20180210/BuildTemplateTabsMenu.png" | relative_url }}" alt="Build Template Tabs Menu"/>
</p>
<p>
- Triggers allow you to configure whether or not to use gated checkins or trigger the build on a checkin or to schedule the build as a nightly or other scheduled build. <br/>
- The Options tab allows you to configure the name and versioning of the build output. <br/>
- Retention specifies how long the assets will be maintained. <br/>
- History will contain references to builds for this template if you have already triggered the build template before.
</p>
<p>
Now that you have a build template you can save or save and queue. The save and queue option will immediately start the dialog to start a build using the template.
<br/><img src="{{"/assets/images/20180210/SaveAndQueueDialog.png" | relative_url }}" alt="Save And Queue Dialog"/>
</p>
<p>
Here you can see the Agent Queue and you can select the hosted queues provided by VSTS or any private build agents that you have configured. Also note the variables, you can change their values here. If you have set up an additional variable in the Variables tab of the build template and you want to set the value at every build, then be sure to check the "Settable on queue time" option
</p>
<p>
After the build has been queued you can see the status under the build tab. Click on the version number link to see the progress and details for a build.
<br/><img src="{{"/assets/images/20180210/BuildsOverview.png" | relative_url }}" alt="Builds Overview"/>
</p> 
<p>
If you are running private agents or multiple builds you might sometimes see the Status as waiting for available agent. If this happens you can check out the agents workload under the Gear icon under the Agent Queue option. You need certain account permissions on the project to access this information and it may not be visible if you do not have permission.
<br/><img src="{{"/assets/images/20180210/AgentQueueOverview.png" | relative_url }}" alt="Agent Queue Overview"/>
</p>
<p>
After a build completes you get an email with the results and you can view the results in the build details after it finishes. 
<br/><img src="{{"/assets/images/20180210/BuildSummary.png" | relative_url }}" alt="Build Summary"/>
</p>
<p>
The resulting output of the build (or rather the artifacts) can be found under the artifacts tab.
<br/><img src="{{"/assets/images/20180210/ArtifactOverview.png" | relative_url }}" alt="Artifact Overview"/>
</p>
<p> 
You could download the artifacts and manually release or publish them or discover the output files of the build in this tab.
</p>