---
layout: post
title: Azure Lighthouse
date: 2020-07-30 13:32:20 +0300
description: Follow along the journey of McCray Projects as we learn new things in information technology and other things like electronics and recycling. # Add post description (optional)
img: blackandblue.jfif # Add image post (optional)
fig-caption: # Add figcaption (optional)
comments: true
tags: [Azure, Lighthouse, Powershell, ARM Template]
---
This post is going to be a little out of the pattern that I have already started as today I am going to talk about Azure Lighthouse.  This product from Microsoft is not on the exam I am studying for but it is something I had to look at.  I remember when this thing first came out and I remember thinking about how complicated it looked.  I attempted to look into it and trying to set it up for my organization but it was just too much for me.  Azure ARM templates, yuck!!


Today, I was asked to take a look at it again and was given an appointment for 2pm to go over it and assist with trying to get it setup.  About 30 minutes before the appointment, I decided to go do some research on it so that I would at least know something about it.  It has been over a year since I have looked at anything related to it.  What came from this appointment is a changed opinion of setting up Lighthouse.  It was so easy.


In short, Lighthouse allows companies who provide Managed Services to other companies to manage those other companies' Azure tenants inside of their own tenant.  No longer would you create users in your customer's tenants and forget to remove them when an employee leaves.  Does your MSP have 100 clients?  Imagine removing a user from each one of those tenants when your employee leaves.  You could alternatively add your employees as guest users to your customer tenants and then assign permissions that way.  Unfortunately though, you will find yourself in the same position because when your employee leaves you still have to remove it from their tenant.  If only you could just add your employees to a group in your tenant and assign that group a role in your client's tenant!!  Now you can!


Let's get down to the real work!  There are two ways to set this up.  You can publish a public offering in Microsoft's Partner Marketplace or you can publish a private offering to your client's tenant.  I chose the second option for this.  We will be figuring out the first option as well but I was having issues with the Microsoft Partner portal.  Who doesn't right?


The first thing you need to decide is what level you need to manage.  Is it a resource group, multiple resource groups, or a whole subscription.  I chose to onboard the entire subscription and to do that I needed two files.  Microsoft provides two templates to use for this and they are called delegatedResourceManagement.json and delegatedResourceManagement.parameters.json. I know what some of you will think and don't worry.  These files are really easy to read and you only have to play with one of them.  Copy both files in to a folder on your C:\ drive called Lighthouse.  Open up the delegatedResourceManagement.parameters.json file in VS Code or your favorite editor.

{% highlight powershell %}
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mspOfferName": {
            "value": "McCray Projects"
        },
        "mspOfferDescription": {
            "value": "Everything Azure!"
        },
        "managedByTenantId": {
            "value": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        },
        "authorizations": {
            "value": [
                {
                    "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
                    "principalIdDisplayName": "Virtual Machine Contributor",
                    "roleDefinitionId": "9980e02c-c2be-4d73-94e8-173b1dc7cf3c"
                },
                {
                    "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
                    "principalIdDisplayName": "Azure Security Reader",
                    "roleDefinitionId": "39bc4728-0917-49c7-9d2c-d95423bc2eb4"
                },
                {
                    "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
                    "principalIdDisplayName": "Azure Reader",
                    "roleDefinitionId": "acdd72a7-3385-48ef-bd42-f606fba81ae7"
                },
                {
                    "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
                    "principalIdDisplayName": "Azure Contributors",
                    "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
                },
                {
                    "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
                    "principalIdDisplayName": "Decommission",
                    "roleDefinitionId": "91c1777a-f3dc-4fae-b103-61d183457e46"
                }                               
            ]
        }
    }
}
{% endhighlight %}
The first two lines can be ignored.  They are required and at this time are not changing. 

Next you have 2 sections that basically provide context to your offering.  The first, mspOfferName, can be thought of as the title and the next one mspOfferDescription is the description.  

On the next one you need to replace the xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your tenant ID and that can be found in Azure Active Directory by going to Properties.

The next section is where all of the meat is at.


You need to decide how you want to distribute roles in your organization.  Maybe you want to just use the default template and do the roles based on Tier 1, Tier 2, Tier 3.  Maybe you want to do it another way.  Which ever method you choose you should create a SECURITY group in your tenant for each grouping and assign the users to those groups.  You can then go to your group in your tenant and on the Overview page you will find the Object ID.  You will need to put that in the "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" section in your file.  You will then need to find the best role to give that group and go to Azure built-in roles and find that role in the table.  You will want to copy the ID from that field into the "roleDefinitionId" section.


So to take from the example above, 

{% highlight powershell %}
{                     
  
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",                     
  "principalIdDisplayName": "Virtual Machine Contributor",                     
  "roleDefinitionId": "9980e02c-c2be-4d73-94e8-173b1dc7cf3c"                 
  
}
{% endhighlight %}

So to take from the example above, this section says that I created a group called "Virtual Machine Contributor" and the ID of that group is "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx".  I am then assigning the built-in role of Virtual Machine Contributor and the ID of that role is "9980e02c-c2be-4d73-94e8-173b1dc7cf3c".  See that was not so hard.  Now obviously I did not make a group in my tenant that has a name matching the role and this is just an example.  You could easily have called it Support Desk and then taken the ID of your Support Desk group.


Once you have completed your groups and have mapped all of the appropriate roles to your groups you will need to save the file.  Then open up Powershell.  I use the Powershell in my VS Code but you can do it however you like.  


First you will need to connect to your client's tenant in Azure as a user with owner permissions of the subscription or have the client connect.  Then you will want to run the following there commands.

{% highlight powershell %}

Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -subscriptionid <subscription id>

{%  endhighlight %}

After you run the first one it will prompt you for log in.  Once you complete the login process, the next command will get a list of available subscriptions.  The final command should be run for each subscription you will need access to after changing the <subscription id> to the actual ID you got from running Get-AzSubscription.


You will then need to type the following command.

{% highlight powershell %}

New-AzSubscriptionDeployment -name "Lighthouse" -location <CLIENT'S  REGION> -templatefile "C:\Lighthouse\delegatedResourceManagement.json" -templateparameterfile "C:\Lighthouse\delegatedResourceManagement.parameters.json" -verbose

{% endhighlight %}

You can change the name to whatever you like and you will want to replace <CLIENT'S  REGION> with the the appropriate value.  Hit enter and that is it.  For me, it was relatively quick for most of them but a few took up to 5 minutes to show up.  


You can validate that it is setup properly by going to your client's tenant and searching for Service Providers.  You should see your newly created offering there.  After a few minutes, you can go to your tenant and search for My Customers.  You should see your client in there and can click through it all the way into the subscription.


If you ever need to make changes to the permissions in the future then just edit the file, save it, and re-run the command.  So easy and yes, you should script that too!  I did, but that's for another post.


There is more!  You know have the ability to automate tasks on all of your clients at one time!  Wish that you could build an Azure Policy and standardize all of your client's tenants?  What other things have you always wanted to do, manage devices from one window?  I'll let you figure that out, but if you have a great idea, let me know and maybe we can figure it out together!