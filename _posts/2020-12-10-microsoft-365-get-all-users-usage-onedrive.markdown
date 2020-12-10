---
layout: post
title: Microsoft 365 Get the usage of OneDrive for all users
date: 2020-11-10 00:00:00 +0300
description: How to use Powershell to get the useage of OneDrive for all users.
img: sharepoint.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
comments: true
tags: [Microsoft, Microsoft 365, Sharepoint, OneDrive, Powershell] # add tag
---

Today, I had an interesting challenge.  We are going to move one of our clients out of Office 365 provided by GoDaddy and in to a normal tenant under our CSP.  During the evaluation of this, we wanted to see how much data everyone is using on OneDrive.  In GoDaddy, you cannot go to the admin portal and go to Reports and select Usage.  The options you normally have are just not there.  I called GoDaddy and they pretty much told me that there is no way to do it in the UI and you must do it through Powershell.  I am now going to show you how.

The first thing you will need is to download and install the Sharepoint Online Powershell module.  Go to [Sharepoint Online Powershell Module](https://www.microsoft.com/en-us/download/details.aspx?id=35588) and install the Powershell module.  You will want to restart Powershell after you have installed it, if you had it open.

You will now want to open Powershell and type in this command changing the "tenantname" to your Sharepoint tenant name.  You can find this by going to the Sharepoint Admin center from the portal.  The URL in the browser will have the value.

{% highlight powershell %}
Connect-SPOService -Url https://tenantname-admin.sharepoint.com
{% endhighlight %}

It should have a pop up the will ask you to log in.  Once you successfully do so, continue.  OneDrive uses Sharepoint as its background storage but you cannot just get the URLs using Get-SPOSite.  They are each unique and there is no one command to get them all.  We are going to run Get-SPOSite and ask it to include the Personal sites and then filter out all of the sites that are not Personal.  We will then just select the URL and place it in the $URL variable.

{% highlight powershell %}
$urls = Get-SPOSite -IncludePersonalSite $true -Limit all -Filter "Url -like '-my.sharepoint.com/personal/'" | Select -ExpandProperty Url
{% endhighlight %}

Now that you have a list of all of the URLs, it is time to get the current usage of those sites in GB and export them to a CSV in the C:\Temp directory. 

{% highlight powershell %}
foreach($url in $urls){
    Get-SPOSite $url -Detailed -ErrorAction SilentlyContinue | select url, owner, @{Name = 'GigsUsed'; Expression = {$_.storageusagecurrent / 1024}} | export-csv c:\temp\forPro3.csv -Append -notypeinformation
}
{% endhighlight %}

And that is it.  You now should have a file at c:\temp\forPro3.csv that contains the URL, owner, and the ammount of storage used in GB.
I hope that this helps!

Thanks,
Carlos