---
layout: post
title: Azure Creating a virtual machine with ARM using Azure Key Vault! Part 1
date: 2020-06-21 00:00:00 +0300
description: So its been a little while!  Life gets busy some times and you just have to deal with that.  Yesterday I was able to get back into it and decided that I would just build 100 VMs in every way possible.  # Add post description (optional)
img: blackandorange.jfif # Add image post (optional)
comments: true
tags: [Azure, Powershell, ARM Template] # add tag
---
So its been a little while!  Life gets busy some times and you just have to deal with that.  Yesterday I was able to get back into it and decided that I would just build 100 VMs in every way possible.   I started with the portal.  I created a VM but going to the marketplace and selecting the Server 2019 template and then selected the defaults on as much stuff as I could.  Not very exciting eh?


I then decided to take a look at the deployment and see more information.  You can do this by going to the Resource Group and then selecting Deployments.  Find your deployment and click on it.  Now you will find 4 separate areas and the one that we are interested in is the Templates.  Although there is a lot of good information in the Overview, Input, and Output areas, we are interested in automation here so off to Templates we go.


I started by looking at the Parameters tab.  I payed attention to the syntax and trying to understand why it is doing what it is doing.  The first thing that I noticed is the first 2 lines.  

{% highlight powershell %}

"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
 "contentVersion": "1.0.0.0"

{% endhighlight %}

These are required.  I am not sure yet how to know when to change the schema but for now I will let that go since it obviously hasn't changed in 5 years.  Maybe this will become important to me today but for now we just need to know that it is needed.  The contentVersion is an area that allows me to keep a versioning control system.  More on that to come.


I decided that I would save these templates to my machine and use them to redeploy.  I saved them locally and then deleted everything in the target resource group.  If I did not remove everything first then the deployment would just validate that it is correct and not deploy anything.  The files downloaded as a ZIP file that contains a template.json and a parameters.json.  I then set out to figure out how I deploy these.  I started in the portal by selecting Template Deployment from the Marketplace.  The options that this gave me didn't seem like this was the place to start so I pulled out the good ole Powershell.  


The first thing that you will have to do is connect to Azure.  Run this command from the Powershell window.

{% highlight powershell %}

connect-azaccount

{% endhighlight %}

You will be directed to a login window or in my case I was directed to a site since I am using Windows Terminal.

{% highlight powershell %}

WARNING: To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code R5DF9PAYX to authenticate.

{% endhighlight %}

Once you complete the login phase you can test by checking to see if you see your Resource Groups.

{% highlight powershell %}

Get-AZResourceGroup

{% endhighlight %}

This is actually an important step.  If you are like me and work for a Managed Service Provider (MSP) then you may be connected to a lot of different tenants.   I had no Resource Groups at all and had to then select a subscription in order to get to the right area.  I ran the following command.

{% highlight powershell %}

select-azsubscription -subscriptionName Pay-As-You-Go

{% endhighlight %}

Once I ran this command I could then look for Resource Groups again and I found my CarlosTraining resource group.


I am now ready to attempt to deploy everything again by using the template.  I did some research and found the following command.  Deploying ARM templates from Powershell requires the New-AZResourceGroupDeployment cmdlet and at least 3 parameters.   You are required to provided the target resource group, the source template.json and the source parameters.json files.  I am using the TemplateFile and TemplateParameterFile since my files are local on my machine.  More on this in part 2 of this post!

{% highlight powershell %}

New-AZResourceGroupDeployment -resourcegroupname "CarlosTraining"
 -TemplateParameterFile C:\CarlosTraining\ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538\parameters.json -TemplateFile C:\CarlosTraining\ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538\template.json

{% endhighlight %}

So when I ran this you would expect that it went flawlessly.  Nope, think again!  Apparently, we need to still supply the admin password!  So as a good little engineer, I went into the parameters.json file and put the admin password.  I then ran the command again and ha ha it worked!  As soon as it was all completed, I deleted everything so that I could go again.  Putting the password in the parameters file is an absolutely HORRIBLE idea so I thought, 


"how do you use a Key Vault for this?"


I went to the portal and searched Key Vault.  I then created a new Key Vault called CarlosTrainingKeyVault in the same region as my VM which is EastUS.  I then added a secret by going into the key vault and selecting Secrets and then selecting Generate\Import.  I called it "admin" and supplied a password.  


I now have to get the parameters.json file to reference that secret in the key vault.  In the parameters.json file I found the "adminPassword" section at the bottom.  I then changed it to read as below.

{% highlight powershell %}

"adminPassword": {
 "reference": {
 "keyVault": {
 "id": "/subscriptions/<subscriptionID>/resourceGroups/CarlosTraining/providers/Microsoft.KeyVault/vaults/CarlosTrainingKeyVault"
                },
 "secretName": "admin"
            }
}

{% endhighlight %}

You will need to change the <subscriptionID> to your subscription ID.  Better yet, if you go to the key vault, you can then go to Properties and the Resource ID can be copied from right there that will have your subscription ID already in it.  You're welcome!  :D


Now save that file and re-run the previous command to deploy the template.

{% highlight powershell %}

New-AZResourceGroupDeployment -resourcegroupname "CarlosTraining"
 -TemplateParameterFile C:\CarlosTraining\ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538\parameters.json -TemplateFile C:\CarlosTraining\ExportedTemplate-CreateVm-MicrosoftWindowsServer.WindowsServer-201-20200627202538\template.json

{% endhighlight %}

If you are like me then you have successfully deployed the same environment twice!  Once via the portal and once via an ARM template!  You also have now learned how to get an ARM template to reference a secret inside of a Key Vault!


This is a lot for today but I did not stop there!  The next obvious place for me to go was to think, 


How would I have done this from a public source like GitHub?


And this is where I leave you for now!  I will go over that question in the next post, Part 2!


See you then!


Carlos
 