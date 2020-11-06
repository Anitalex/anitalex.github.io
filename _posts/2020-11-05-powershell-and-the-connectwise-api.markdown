---
layout: post
title: Powershell and the ConnectWise API
date: 2020-11-05 00:00:00 +0300
description: Using the ConnectWise API with Powershell to do thing such as opening tickets and adding time.
img: workflow.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
comments: true
tags: [Azure, DevOps, ConnectWise, Powershell] # add tag
---

I now have been working with ConnectWise for over 4 years and during this time I have also managed project teams.  One of the big issues that we have in a Managed Service Provider (MSP) world is running projects out of ConnectWise.  Our teams favor the Agile project management method and we would rather run our projects that way but ConnectWise is just not setup for that.  The delima my teams have had is that ConnectWise is the source of record for the business and to use another tool would just mean more things to keep track up and possibly double entry of time and notes.  I have always wanted to use another tool such as Microsoft Planner, [Asana](https://asana.com/)[, [Basecamp]([https://basecamp.com/) but more recently we are looking at using Azure DevOps.  I have been playing with the ConnectWise API off and on during this time and the other day I finally cracked the code, so to speak!

The first thing that you will need to is to go to ConnectWise Developer at https://developer.connectwise.com/ and register.  You will then need to create a Partner Client ID and ensure you use the appropriate Integration Type for your server.

![ConnectWisePartnerID](/assets/ConnectWisePartnerID.PNG)

Once your request is approved you will get an ID similar to xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx an you will need this shortly.  The next thing that you will need to do is to go into your ConnectWise and create an API key.  To do this you will need to be an admin or have someone who has the permissions to help you.  On the bottom left of ConnectWise is a System icon.  When you hover over that you will get a pane open up and you will want to select Members.  Next, you will want to go to the API Keys tab and create an API key and ensure that you record the private key as you will never see it again.

At this point, you should now have a Partner Client ID from https://developer.connectwise.com/ and a public and private key from your instance of ConnectWise.

For this example, I am going to make up a company called SteveIT that has a domain of SteveIT and this company has their own ConnectWise server at cw.steveIT.com.  I checked and at this time that address is not routable so I think we are good.  SteveIT also has a call center that works off a ConnectWise board named HelpDesk. I am going to make up some keys to show what the look similar to.  You will need to replace these values with your own. 

{% highlight powershell %}
$Company = 'steveIT'
$PubKey = 'hdoe65les95jsw0' 
$PrivateKey = 'f6d7rjeks93kd83' 
$ClientID = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' 
{% endhighlight %}

I am going to create an encoded authentication which basically will encode the company, public key, and private key into a BASE 64 string.  This will look something like this cHJvhdelW4rRktEaHlidDMzkdyVCwouthglGtnN2x5SllLZE0wie76 and will place the value in the $EncodedAuth variable

{% highlight powershell %}
$AuthString  = "$($Company)+$($PubKey):$($PrivateKey)"
$EncodedAuth  = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes($AuthString));
{% endhighlight %}

We will then have to build our headers that we will send over when connecting to the REST API.  We will set the Authorization to Basic using the encoded authentication string.  We will supply the Partner Client ID and set the connection method to Key.  We will also provide an Accept string.  

{% highlight powershell %}
$Headers = @{
    Authorization = "Basic $EncodedAuth"
    ClientID = $ClientID
    'Cache-Control'= 'no-cache'
    ConnectionMethod = 'Key'
    Accept = "application/vnd.connectwise.com+json; version=v2020_2"
}
{% endhighlight %}

This is actually one key area that was one of the reasons that it took me so long to get this to work.  I had tried using a module called [PSConnectWise([https://github.com/sgtoj/PSConnectWise)] which is really well developed but it appears to be really aging.  When I tried this module it would not work for me due to the Accept string in the module being "application/vnd.connectwise.com+json;" and my version of ConnectWise was requiring the version.  I found the place in that module where this line was and tried to add "version=v2020_2" but that also failed.  This was the reason I had to find a way to write my own.  The other reason that it took me so long was that I did not realize the need of having the Partner Client ID and the public and private keys.

The last piece of information that you will need to provide is the URL and the Body.  The URL will need to contain the path to the object that you will want to retrieve or place.  You can check the API reference to get a list of all of these paths.  I will also post a few below.  As you can see in this instance I am going to be looking for a service ticket.  The Body needs to contain your conditional logic to find what ever it is you are looking for.   In my example, I am looking for open tickets on the HelpDesk board. 

{% highlight powershell %}
$URI = "https://cw.steveIT.com/v4_6_release/apis/3.0/service/tickets"
$body = @{
    "conditions" = "closedFlag = False and board/name='helpdesk'"
}
{% endhighlight %}

Now that you have everything that you need, you will call Invoke-WebRequest providing the URL, Body, and Headers along with the REST Method that you will use.  This should find all tickets on the HelpDesk board and place them as JSON in the variable $results.  I have chosen to display my results in a grid view so I take the content and convert it from JSON and pipe it to Out-GridView.

{% highlight powershell %}
$results = Invoke-WebRequest -Uri $URI -Method 'GET' -Headers $headers -Body $body
$results.content | ConvertFrom-Json | Out-GridView
{% endhighlight %}

You should now have a box that has popped up that has a list of tickets on your board.

Here are a few other examples that I have to help do other things in ConnectWise.

This is an example that will get a list of open projects in the Execution status on the Project portal board called 'projects'.  Notice that the URL changed to look for a different type of object.

{% highlight powershell %}
$URI = "https://cw.steveIT.com/v4_6_release/apis/3.0/project/projects"
$body = @{
    "conditions" = "status/name like '%Execution%'and board/name='projects'"
}
$results = Invoke-WebRequest -Uri $URI -Method 'GET' -Headers $headers -Body $body
$results.content | ConvertFrom-Json | Out-GridView
{% endhighlight %}

This example will get you a list of project tickets on open projects on the Project portal board called 'projects'

{% highlight powershell %}
$URI = "https://cw.steveIT.com/v4_6_release/apis/3.0/project/tickets"
$body = @{
    "conditions" = "board/name='projects' AND closedFlag = False"
}
$results = Invoke-WebRequest -Uri $URI -Method 'GET' -Headers $headers -Body $body
$results.content | ConvertFrom-Json | Out-GridView
{% endhighlight %}

This example will create a ticket on the HelpDesk board in the status of New with a type of Network.  Your Types, Subtypes, and Items in ConnectWise may differ.  You will need to get the ID of the company that you want to make the ticket for and for the contact on the ticket.  You will also need to get the ID of the priority and source that you want to use.  One thing you will notice here about the Invoke-WebRequest is that we had to add ContentType to it for it to work.  Notice that also the REST method changed to POST and the URL changed again.

{% highlight powershell %}
$Summary = "My computer will not boot"
$Body= @{
    "summary"   =    "$Summary"
    "board"     =    @{"name" = "helpdesk"}
    "status"    =    @{"name" = "New"}
    "company"   =    @{"id" = "2"}
    "contact"   =    @{"id" = "65489"}
    "Type"     =    @{"Name" = "Network"}
    "Priority"     =    @{"id" = "4"}
    "Source"     =    @{"id" = "2"}
}

$Body = ConvertTo-Json $Body
[string]$ContentType = "application/json"
$URI = "https://cw.steveIT.com/v4_6_release/apis/3.0/service/tickets"
$results = Invoke-WebRequest -Uri $URI -Method 'POST' -Headers $headers -Body $body -ContentType $ContentType
$results.content | ConvertFrom-Json | Out-GridView
{% endhighlight %}

This example will show you how to enter time into ticket number 123456 with a specific start and end date and time.  Inside of the Body we have to convert this date and time to UniversalTime.  This was required and it did not work without it.  You will also need to get IDs for the Location and Business Unit.  This Location is not the client's location but rather the Location in the Overview area of a time entry.  Make sure that you replace your other items like Work Type and Work Role to match your environment.

{% highlight powershell %}
[int]$ticket = 123456
$start = '10/22/2020 7:30:00 AM'
$end = '10/22/2020 7:46:13 AM'
$notes = "I entered this time with Powershell"
$Body= @{
    "ChargeToType"      =    "ServiceTicket"
    "locationId"        =    2
    "ChargeToID"        =    $ticket
    "company"           =    @{"identifier" = "ABC"}
    "member"            =    @{"identifier" = "cmccray"}
    "worktype"          =    @{"name" = "Remote Support"}
    "workrole"          =    @{"Name" = "Engineer"}
    "businessUnitId"    =    8
    "timeStart"         =    $start.ToUniversalTime()
    "timeEnd"           =    $end.ToUniversalTime()
    "billableOption"    =   "DoNotBill"
    "notes"             =    $notes

}
$Body = ConvertTo-Json $Body -Depth 100
[string]$ContentType = "application/json"
$URI = "https://cw.steveIT.com/v4_6_release/apis/3.0/time/entries"
$results = Invoke-WebRequest -Uri $URI -Method 'POST' -Headers $headers -Body $body -ContentType $ContentType
{% endhighlight %}

This example gets a list of time entries after 7am on 10/22/2020.  Notice here that the URL has been changed and the REST method is back to GET

{% highlight powershell %}
$URI = "https://cw.steveIT.com/v4_6_release/apis/3.0/time/entries"
$body = @{
    "conditions" = "timestart > [10/22/2020 7:00:00 AM]"
}
$results = Invoke-WebRequest -Uri $URI -Method 'GET' -Headers $headers -Body $body
$results.content | ConvertFrom-Json | Out-GridView
{% endhighlight %}

If you are also trying to work with your ConnectWise environment using Powershell, I hope this helps you!  Feel free to let me know if you have any issues and I will do my best to help where I can!

Thanks
Carlos