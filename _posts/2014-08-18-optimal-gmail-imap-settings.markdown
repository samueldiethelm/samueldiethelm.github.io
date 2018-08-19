---
layout: post
title:  "Optimal Gmail IMAP Settings"
categories: gmail mac
---
I am writing this post as I have spend most of the day trying to setup Mavericks Mail to work with Gmail properly. I have been using OSX with Gmail for a long time now, but recently I noticed Mail.app on my MacBook Air was consuming too much CPU at times and my iMac was getting unresponsive too as I could hear the hard disk constantly spinning. I noticed after some time that it happened while syncing with gmail servers.<!--more-->

![Gmail Activity](http://www.samueldiethelm.com/wp-content/uploads/2014/01/gmail_activity.png){: .center-image }

I did some research and noticed that what was causing the issue was  the multiple "labels" I had in place in Gmail. To be compliant with IMAP, those labels are converted into folders, each of them containing all the messages with the given label. This is a problem when you use multiple labels, and some emails get more than one label, as the email will get saved on your computer duplicated as many times as labels it has, plus once for the "all mail" folder and once more for the regular inbox folder.

To solve this issue, I went into Gmail settings, into the [labels](https://mail.google.com/mail/u/0/#settings/labels "Google Label Settings" ) tab. At  the top you see "System labels" but I will get to those. At the bottom of this page you see a list of all the labels and how they behave with respect to IMAP. My advice is to uncheck all labels from "Show in IMAP". The only problem with that is you won't see your emails sorted into pretty folders in Mail.app, but rather in the inbox.

About the "System labels" mentioned above, I recommend unchecking all of them from"Show in IMAP" except Inbox, Starred, Sent Mail, All Mail and Trash. All of those should be on at least:

* **Inbox**, for obvious reasons, as you want your email to be delivered to your computer.
* **Starred** I would keep on if you make use if "flags" in Mail.app as all flagged items are moved "starred" on gmail servers and if disabled, it can behave strangely. If not, you shouldn't have to bother.
* **Sent mail**, again as Inbox, so that sent mails are safely stored on on the Sent folder. You should get an alert from Mail.app if you disable it anyway.
* **All mail**, for emails you click to be "archived" in Mail.app.
* And finally **Trash** so that deleted emails go the appropriate place.

I should also mention that if you uncheck "Starred" and you did have something starred before doing so, those emails won't show under "Flagged" in Mail.app, but it will get confused and the counter on the right side of "Flagged" might stay with the count of the amount of starred elements there were in there before.

&nbsp;
