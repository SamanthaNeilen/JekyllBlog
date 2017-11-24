---
layout: post
title:  "Starting my blog"
date:   2017-11-18 08:46:41 +0100
tags: Blog Jekyll
---
<p>
	I would like to start my blog with a post of how I set up this blog. 
<p>
<p>
	When you start a Github account you can use a repository with the name username.github.io to host a website. That website will be hosted at the url <a href="#">https://username.github.com</a>. These websites on Github are called Github Pages. You can also create a website for a specific repository. Seeing as you can create a free Github account as long as you don't want or need private repositories, this is a great way to host a blog or website for free. For more information see <a href="https://pages.github.com/" target="_blank">Github Pages</a>. 
</p>
<p>
	Github pages promotes Jekyll as a tool to build static websites. It's not officially supported from windows but so for installing it and running it on my Windows 10 (2017 Fall Creators Update) has worked out fine so far.
	See <a href="https://jekyllrb.com/" target="_blank">the Jekyll website</a> for installation instructions.
</p>
<p>
There were a few points in the installation that I had to look up because of my unfamiliarity with Linux so I've included a description below of the steps taken after installing the Ubuntu store app from windows.
</p>
<h3>Installing Jekyll on windows10</h3>
<p>
After I installed the Ubuntu app and started it as an Administrator I got an error. The link in the error shows you to the MSDN page explaining that you have to run a Powershell command to actually enable the Linux subsystem for Windows.
For more information see: <a href="https://msdn.microsoft.com/en-us/commandline/wsl/install-win10" target="_blank">Install the Windows Subsystem for Linux</a>.
</p>
<p>
The command that needs to be run (as administrator in Powershell) was:
{% highlight shell %}
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
{% endhighlight %}
</p>
<p>
	After the reboot start the Ubuntu terminal app again as administrator. It will now show the screen below:<br/>
	<img src="{{"/assets/images/20171118/UbuntuInstallingMessage.png" | relative_url }}" alt="Ubuntu Installing Message"/>
</p>
<p>
The terminal actually froze up for me the first time here, so if it's not continuing after a few minutes, just close and restart the app. (This may happen a few times when configuring the shell or downloading and updating the packages needed to install Jekyll. Also sometimes the editor seems to not take keyboard input when prompted to press enter or cntrl-c, when that happens click a few times within the editor to get the focus back correctly in the terminal and it will accept the command.)
</p>
<p>
When it's done, you will see an active command line with root@computername:~#<br/>
Enter the command below to start bash (the Jekyll website just stated bash without the sudo keyword but the sudo keyword is needed).
{% highlight shell %}
sudo bash
{%endhighlight%}
</p>
<p>
It may prompt you for a username and password. (Be aware you will need that password every time you start up bash. Also the text cursor will not show any signs off accepting input like stars or underscores but it will register your keystrokes). I have installed the terminal on 2 different computers. On one I did not get prompted for the separate login on the other I did need it.
</p>
<p>
Run the commands as provided on the Jekyll site for <a href="https://jekyllrb.com/docs/windows/" target="_blank">the installation on windows</a>. When it shows multiple commands in one code block run them per command and don't paste them in all at once). 
</p>
<p>
Also it may give the message below. You can continue by pressing enter and run the next command when it finishes. <br/>
<img src="{{"/assets/images/20171118/BrightboxRubyMessage.png" | relative_url }}" alt="BrightBox Ruby Message"/>
</p>
<p>
After all the commands have run and Jekyll is returning a version number you can navigate to a windows folder to create your first Jekyll website. The Ubuntu terminal is run from a separate sub filesystem so you need to enter the command below to navigate to your C:\ drive. 
{% highlight shell %}
cd /mnt/c/
{%endhighlight%}
</p>
<p>
From the C:\ directory you can just use the "dir" and "cd" commands to navigate to the appropriate folder the same way you would in a Windows command prompt. To create a new Jekyll website within your current folder use the command below. (note that you do not need to use the sudo keyword any more.)
{% highlight shell %}
jekyll new .
{%endhighlight%}
</p>
<p>
After it has finished you have a working default website that you van run with the command below.
{% highlight shell %}
jekyll serve
{%endhighlight%}
</p>
<h3>Customising your site</h3>
<p>
You can now add pages like the one already generated in your _posts folder and alter the _config.yml file to populate the contents of your generated site.
</p>
<p>
Every time you make a change when Jekyll serve command is active or when you issue a Jekyll build command the contents of the _site folder get updated with the generated HTML site that you can access with your browser. This is also the content you can upload to your Git repository and thus will be hosted on your username.github.io website.<br/>
<img src="{{"/assets/images/20171118/JekyllDefaultPage.png" | relative_url }}" alt="Jekkyl Default Page"/>
</p>
<p>
As you may notice the default site is quite barren but you do not see the files for the page layouts or the stylesheets. These files can be found in the folder that holds the default theme. It was installed with the other bundles in the Ubuntu terminal. As stated in the <a href="https://jekyllrb.com/docs/structure/">Jekyll documentation</a> ( you can find out where those files are located with the command below.
{% highlight shell %}
bundle show minima
{%endhighlight%}
</p>
<p>
You can copy the files from the theme folder and copy them into the folder of your Jekyll page to completely customize them. However the files installed by the Linux terminal are installed in a folder in your AppData folder. To access them from a windows explorer you need to enable hidden folders in your folder options and search for the "gem" in your %Users%\User\AppData folder to locate the installed ruby packages (including the minima package)
</p>
<p>
Some great references on how to customize your pages are:
<ul>
	<li>
	<a href="https://jekyllrb.com/docs/templates/" target="_blank">Jekyll template documentation</a>
	</li>
	<li>
	<a href="https://shopify.github.io/liquid/" target="_blank">Liquid template language</a>
	</li>
	<li>
	<a href="https://gist.github.com/smutnyleszek/9803727" target="_blank">Liquid for Github pages cheatcheat</a> 
	</li>
</ul>
</p>
<p>
After customizing your site template you can just worry about creating content pages. Any changes you make to your template at a later time will be reflected over all pages in the site. When uploading changes to your site repository, it may take a minute or so before changes are reflected in the browser.
The files used to generate the content for this blog can be found at <a href="https://github.com/SamanthaNeilen/JekyllBlog" target="_blank">https://github.com/SamanthaNeilen/JekyllBlog</a>.
</p>