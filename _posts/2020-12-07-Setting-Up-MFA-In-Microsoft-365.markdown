---
layout: post
title: Setting up MFA in Microsoft 365
date: 2020-12-07 00:00:00 +0300
description: Turning on MFA in Microsoft 365
img: lock.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
comments: true
tags: [Azure, Microsoft 365, MFA] # add tag
---

Today security has got to be one of, if not the hottest topic in business (next to Covid19, of course).  We seem to have a new article every week where some company got hacked and their client's information is now being sold to the highest bidder on the dark web.  These breaches are happening all the time and the techniques used to start the breaches are moving more and more out of the technical realm and are now social as many people probably do not realize that they are the new target.  They see the headlines and think that some person working in IT made a mistake again and now their company is going to pay for it.  The sad part is that while they think that someone else did something wrong, they are going through their lives doing the same things, blissfully unaware that they too could be next because the like to do the same things that everyone does.  Reuse passwords.

Let me tell you how they do this.  They get their hands on a password list and these lists are usually just usernames and passwords that have been collected by other breaches.   If you have not ever been to [HaveIBeenPWNed](https://haveibeenpwned.com/Passwords) before then head over there and check to see if your email or passwords have been breached.  If so, I recommend you immediately change it.  This site was created so that people can check to see if they were part of a breach in the past.  They then will take all of your passwords that are found from these dumps and attempt them on all of the top sites that they can find.  You see the problem is that everyone reuses passwords all the time.  If I get your password from one place, then I can easily check all the other sites to see if it works there too.  If the password is successful when trying to get in to your email with this method, then it is now in the final minutes of game over for you and the only thing stopping the game from ending right now is what happens next.  Do you have multi-factor authentication turned on?

Multi-factor Authentication (MFA) means that you have to have more than one method of authenticating that you are who you say you are.  Almost everyone has some account out there where after putting in their password, the site sends you a text message with a code that you have to use.  This is called multi-factor authentication.  You must Know the password and Have the phone to properly authenticate to the site.  Although this is not a perfect system, it certainly is one better than just knowing the password of your email. 

I would like to show you how to protect your email in Microsoft 365 with multi-factor authentication (MFA) so that you can add this layer of defense to yourself or your company.  

Go to [Office Portal](https://portal.office.com) and log in.  If this does not take you to the Admin section, then go to the 9 dots in the top left and click it to find Admin.  If you do not have this then you do not have the permissions and will need to find out who does.  

![M365AdminIcon](/assets/img/M365AdminIcon.jpg)

This will take you to the Admin portal of Microsoft 365.  From here you will want to click on Users on the left pane and then Active Users.

![M365ActiveUsers](/assets/img/M365ActiveUsers.jpg)

You should now see a list of users that you have in your portal.  There is a menu bar that goes across the top just above the list of users.  Click on Multi-factor Authentication.

![M365Multi-factor](/assets/img/M365Multi-factor.jpg)

This should have taken you to a new page that provides you a list of all your users again but looks a little different than before.  It should now have a 3rd column that shows the status of multi-factor authentication for each user in your portal.  If you click on one of the users, it will then provide you another pane to the right that has some Quick Steps.

![M365MFAQuickSteps](/assets/img/M365MFAQuickSteps.jpg)

Click on Enable to turn on multi-factor authentication for that user.  When you do this, you will notice that it changes to Enabled.  You will now need for that user to go to a browser and type in www.office.com.  When they go to that site, they will want to log in with their credentials for the account you just enabled MFA on.  Once they put in the correct password it will walk them through adding their cell phone to their account and enabling MFA using either a text message or an application installed on the phone such as Microsoft Authenticator.  

Once they have completed this step go back to the portal to see that the status of Enabled has been changed to Enforced.  Once all of the users in your portal have Enforced next to them then you will be protected by MFA.  A little disclaimer though, some accounts in your portal may be used for services like applications or scanning.  You may not want to enable MFA on these as it may break that service.  Additionally, I suggest to you that you make one account with Global Administrator role that you will NEVER use and set its password to never expire and do not enable MFA on the account.  You will want to store this account for the single purpose of if you accidentally lock all admins out and cannot get back in.

If you have hundreds of accounts and enabling them manually is not something that seems like a good time, reach out and we can help you take care of that or look out for a future article about it. 

Thanks 
Carlos