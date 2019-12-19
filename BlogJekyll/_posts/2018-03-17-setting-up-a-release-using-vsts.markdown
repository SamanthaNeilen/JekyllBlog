---
layout: post
title:  "Setting up a release using VSTS"
date:   2018-03-17 00:00:00 +0100
tags: DevOps
---
<p>
    In <a href="https://samanthaneilen.github.io/2018/02/10/setting-up-a-build-using-vsts.html" target="_blank">my last</a> post, I showed how to build your project using VSTS. In this blog post, I will describe how to set up a release to deploy the database and website from that build to an Azure Environment. 
</p>
<h3>Preparing the project for the release environment</h3>
<p>
    First I would like to point out some changes I made to the ECommerceApp. First was the Target platform of the database project property pages.
<br/><img src="{{"/assets/images/20180317/DatabaseProjectTarget.png" | relative_url }}" alt="Database Release Target"/>
</p>
<p>
    It needs to be set to target an Azure SQL Database as that will be my target platform. Not setting this to the correct target will lead to errors when deploying the .dacpac file. Next, I altered the Web.Release.config. The transformation rules below will change any local paths to HIDDEN.
<br/><img src="{{"/assets/images/20180317/ReleaseConfig.png" | relative_url }}" alt="Release Config"/>
</p>
<p>
    These settings will be overwritten by the settings in the "Application settings" configured in the Azure Portal on the Web App, so there is no need for those paths to be filled with Debug or local settings. In production environments, this would ensure no that no sensitive data is in the web.config on the file system and only in the “Application Settings” on the portal. The settings in the portal can be secured with access rights and are saved encrypted in the Azure Environment. The project contains a file download that expects a download to a local folder. That will not work on Azure, so I added the EnableExport feature flag that is set to false in the deployment.
</p>
<h3>Setting up the Azure artifacts for deployment</h3>
<p>
    In my example, I set up de Web app and the Azure SQL Database by hand. The deployment pipelines in VSTS have steps to create these artifacts during a release if they do not exist with PowerShell steps. This is beyond the scope of this blog post. In this post, I assume that the Web app and Azure SQL Database are present and correctly configured in the Azure environment that I am targeting. My resource group looks like the image below:
<br/><img src="{{"/assets/images/20180317/ResourceGroupAzure.png" | relative_url }}" alt="Resource Group Azure"/>
</p>
<p>
    Also, my Web App already has the correct application settings and connection string configured in the Application settings tab as shown below. The application and configuration settings in the Azure portal are used at runtime instead of the settings in the web.config that is deployed in the Azure environment.
<br/><img src="{{"/assets/images/20180317/ApplicationSettingsAzure.png" | relative_url }}" alt="Application Settings Azure"/>
</p> 
<h3>Setting up the release</h3>
<p>
    In your VSTS project go to the Build and Release menu, then click Releases and then select New definition.
<br/><img src="{{"/assets/images/20180317/CreateReleasePage.png" | relative_url }}" alt="Create Release Page"/>
</p>
<p>
    In the new Release Definition window select, I selected the “Empty” template and named my deployment environment Test.
<br/><img src="{{"/assets/images/20180317/NewReleaseDefinitionOverview.png" | relative_url }}" alt="New Release Definition Overview"/>
</p>
<p> 
    Next, I want to select the output artifacts (the database .dacpac and web deployment zip) from my build with the Add button next to Artifacts.
<br/><img src="{{"/assets/images/20180317/ArtifactSelection.png" | relative_url }}" alt="Artifact Selection"/>
</p>
<p>
    After selecting your artifacts the build trigger will become available. You can set the trigger to create the release every time a build has been successfully created.
<br/><img src="{{"/assets/images/20180317/DeploymentTrigger.png" | relative_url }}" alt="Deployment Trigger"/>
</p>
<p> 
    Next are the pre-deployment settings where you can set up extra rules that the deployment will start after a manual approval by for example your QA team.  I kept the defaults for this step.
<br/><img src="{{"/assets/images/20180317/PreDeploymentConditions.png" | relative_url }}" alt="Pre-Deployment Conditions"/>
</p>
<p>
    Next click on the 1 phase, 0 tasks link to get into the screen where you can actually add the deployment steps for the environment. Here add an Azure SQL DacpacTask and an Azure App Service Deploy task. Then set up the tasks to deploy to your Azure environment.
<br/><img src="{{"/assets/images/20180317/EnvironmentSteps.png" | relative_url }}" alt="Environment Steps"/>
</p>
<p>
    The DacpacTask is configured as shown below:
<br/><img src="{{"/assets/images/20180317/DacPacStepDetails.png" | relative_url }}" alt="DacPac Step Details"/>
</p>
<p>
    The Azure App Service Deploy is configured as below. I only set up the settings shown below and used the defaults for all the others. Notice the menu to deploy or change the application and configuration settings. As all my configuration is already set up correctly in the portal, I do not need to set these up here.
<br/><img src="{{"/assets/images/20180317/WebAppStepDetails.png" | relative_url }}" alt="WebApp Step Details"/>
</p>
<p>
    These steps are enough to deploy the output from the build to the Azure environment. You can return to the release overview using the Pipeline option and add some post-release options to the environment. Post-deployment approvals can be used to sign off on the release after the deployment has been tested. So if you add another environment like Acceptance or Production after the Test environment (using the Add button) the Post-Deployment approval is used to halt the release for the next environment until the current environment is approved by the QA team. 
<br/><img src="{{"/assets/images/20180317/PostDeploymentConditions.png" | relative_url }}" alt="Post Deployment Conditions"/>
</p>
<p>
    In the options tab of the release definition, you can set up the format for the naming conventions for your release.
<br/><img src="{{"/assets/images/20180317/ReleaseOptions.png" | relative_url }}" alt="Release Options"/>
</p>
<p>
    Now that we’ve set up the release pipeline, save it and start a new build to trigger the release. As I did not set any pre-deployment approvals the build will immediately and automatically release to the test environment after the build is complete.
</p>
<p>
    You can see the active releases under the release tab and also go to the overview or to the release details to execute approval actions on an environment.
<br/><img src="{{"/assets/images/20180317/ReleaseOverview.png" | relative_url }}" alt="Release Overview"/>
</p>
<p>
    After the release has been completed you can see an overview by clicking on the name of a release.
<br/><img src="{{"/assets/images/20180317/ReleaseDetails.png" | relative_url }}" alt="Release Details"/> 
</p>
<p>
    If a deploy fails click on the log tab in the release details to troubleshoot errors.
<br/><img src="{{"/assets/images/20180317/ReleaseDetailsLogs.png" | relative_url }}" alt="Release Details Logs"/>
</p>
