---
layout: post
title: Powershell and the Proofpoint API
date: 2020-12-08 00:00:00 +0300
description: Using the Proofpoint API with Powershell.
img: email.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
comments: true
tags: [Azure, DevOps, Proofpoint, Powershell] # add tag
---

I have been working in IT projects now for 5 years and doing the same thing over and over can get a little boring and mundane.  That's why my team and I have been working hard to automate a lot of the projects that we do.  One project in particular is that we implement Proofpoint for clients who want it.  We have already done some scripting on this but were not able to get the API working to be able to make changes in there.  Here is what was figured out.

The script below will get a list of users for a particular domain.  first thing that we need to do is to setup some variables.  We will need to provide a username and password of an account that has Channel Admin permissions to if you would like to create or update Organzations or Organization Admin if you would like to create or update End Users or Silent Users within their organization.  I performed my testing with a Channel Admin and was able to see my client's users with this.  The next thing that you will need is their domain such as Google.com.

{% highlight powershell %}
$username = Read-Host "What is the username" 
$password = Read-Host "What is the password" 
$domain = Read-Host "what is the domain"
$URI = "https://us3.proofpointessentials.com/api/v1/orgs/$domain/users"
{% endhighlight %}

Before we can connect we will need to build the Headers.  This is where you will send the credentials over as values to X-User and X-Password.

{% highlight powershell %}
$Headers = @{
    'X-user' = $username
    'X-password' = $password
}
{% endhighlight %}

Now we can use Invoke-WebRequest to call the REST method of GET to Proofpoint's API and send the credentials in the Header.  The results of calling the API are placed in the $results variable.

{% highlight powershell %}
$results = Invoke-WebRequest -Uri $URI -Method 'GET' -Headers $headers
{% endhighlight %}

Lastly here is some code to output those users to GridView so that they are easier to read.  At this point though, you could do whatever you wanted with them.

{% highlight powershell %}
($results.content | Convertfrom-Json).users | Out-GridView
{% endhighlight %}

I hope this helps if you are trying to do the same.  You can manipulate other objects simply by using the API documentation to find the right URI to send for the object or by changing the REST method to something that makes changes like PUT, PATCH, or POST


Thanks
Carlos