---
layout: post
title: Azure Creating a virtual machine with ARM using Azure Key Vault! Part 2
date: 2020-06-28 00:00:00 +0300
description: In the previous post, Part 1, we covered creating a virtual machine using an ARM template and storing the admin password in an Azure Key Vault.  Once I got this working, I thought that it would be cool if I could put the files in to version control and run them from there.  I already have a GitHub account that I have p. # Add post description (optional)
img: ladydatacenter.jfif # Add image post (optional)
comments: true
tags: [Azure, Powershell, ARM Template] # add tag
---

In the previous post, Part 1, we covered creating a virtual machine using an ARM template and storing the admin password in an Azure Key Vault.  Once I got this working, I thought that it would be cool if I could put the files in to version control and run them from there.  I already have a GitHub account that I have played with in the past so I downloaded and installed Git.  


I opened Powershell and installed Chocolatey by getting the install command from their website at chocolatey.org.  Chocolatey is great because it works like Aptitude on Linux where you can install, upgrade, update and remove applications from the command line.  Once you install Chocolatey you will need to close Powershell and re-open it so that it can reload the environmental variables and the path will work.  Once you do this run the following command.

{% highlight powershell %}

choco install git --params "/GitAndUnixToolsOnPath /NoGitLfs /NoAutoCrlf" 

{% endhighlight %}

This will install Git.  Again you will need to close and reopen Powershell to reload the environmental variables so that the paths work.  Once you do that you will want to create a local folder to store everything in.  In my case, I use C:\GitHub.  Navigate to that folder in the command line and type the following command.


{% highlight powershell %}

git init

{% endhighlight %}

This will initialize git to the folder.  I then used this command to clone my repo to the local folder.

{% highlight powershell %}

git clone https://github.com/Anitalex/Azure.git

{% endhighlight %}

This will then copy my Azure repo to the local GitHub folder.  I now have a path of C:\GitHub\Azure.  I now copied my previous working folder of CarlosTraining into the Templates folder that I already had inside of C:\GitHub\Azure.  I now need to commit all of these changes.  I start by adding the files with this command.

{% highlight powershell %}

git add .

{% endhighlight %}

I then commit the changes using this command

{% highlight powershell %}

git commit -m "Initial Commit"

{% endhighlight %}

I now have my changes ready to be pushed to GitHub.  I now use the following command to push them to GitHub.

{% highlight powershell %}

git push origin master

{% endhighlight %}

Now the files in my GitHub account are the same as my working directory at C:\GitHub\Azure.  So now I will have to figure out how to use this to deploy the VM in Azure.

I know that before I used the -TemplateFile and -TemplateParameterFile parameters with the New-AZResourceGroupDeployment cmdlet to deploy the files but they are only used for local files.  The new parameters for running from a URL are -TemplateURI and -TemplateParameterURI.  I changed them to use the URLs paths of the file in my GitHub repo as shown below.

{% highlight powershell %}

New-AZResourceGroupDeployment -resourcegroupname CarlosTraining -TemplateURI "https://github.com/Anitalex/Azure/blob/master/Templates/CarlosTraining/ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538/template.json" -TemplateParameterURI "https://github.com/Anitalex/Azure/blob/master/Templates/CarlosTraining/ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538/parameters.json"

{% endhighlight %}

When I ran this command, I immediately get the error below.

{% highlight powershell %}

New-AzResourceGroupDeployment: 12:22:32 PM - Error: Code=InvalidTemplate; Message=Deployment template parse failed: 'Unexpected character encountered while parsing value: <. Path '', line 0, position 0.'.

New-AzResourceGroupDeployment: The deployment validation failed

{% endhighlight %}

I did a little research and found that this error happens because GitHub adds some information to the data that is sent and causes it to error out.  In order to get this to work you need to go to the file in GitHub and then click the Raw button.  You can then save the URL from there.



I have now replaced the command with the raw links to the files as shown below.  

{% highlight powershell %}

New-AZResourceGroupDeployment -resourcegroupname carlostraining  -templateuri "https://raw.githubusercontent.com/Anitalex/Azure/master/Templates/CarlosTraining/ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538/template.json"   -templateparameteruri "https://raw.githubusercontent.com/Anitalex/Azure/master/Templates/CarlosTraining/ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538/parameters.json"

{% endhighlight %}

When I run this command it now works like a charm! Now you can edit the files on your local machine and commit them to GitHub.  Then you can run the Powershell command to deploy the templates from GitHub.  I personally us Visual Studio Code as my IDE and I was able to do all of this from there.  Now I can make changes to my parameters.json and hit CTRL+S to save it.  Then in VS Code, I type following command and my new changes get deployed to Azure.

{% highlight powershell %}

git add .
git commit -m "Some message about changes"
git push origin master
New-AZResourceGroupDeployment -resourcegroupname carlostraining  -templateuri "https://raw.githubusercontent.com/Anitalex/Azure/master/Templates/CarlosTraining/ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538/template.json"   -templateparameteruri "https://raw.githubusercontent.com/Anitalex/Azure/master/Templates/CarlosTraining/ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538/parameters.json"

{% endhighlight %}

Are you wondering the same thing that I am wondering?  That's right!  What if I don't have to do this last step?  What if all I had to do is push my changes to GitHub and then everything happens from there?  I think I see a Part 3 coming on!


For now though, have a wonderful day!


Carlos
