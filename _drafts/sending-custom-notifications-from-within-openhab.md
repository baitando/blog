---
layout: post
title:  "Sending Custom Notifications from within openHAB"
category: IT
tags: openhab smart-home
bigimg: /img/bigimg_notification.jpeg
credname: Snapwire
credurl: https://www.pexels.com/photo/beach-bottle-cold-daylight-292426/
comments: true
excerpt: There are several types of notifications supported by openHAB. This post shows how to send notifications via mail, via Telegram and via app.
---

Besides the well known binding extensions of openHAB there are several extensions available for sending notifications from within your smart home environment. In this post I would like to pick some and show how to configure and use them.

We will start with some general topics, which are common for all of the selected notification extensions. With this knowledge in mind, we will have a closer look at how to send notifications via the openHAB app, classic mail, and the instant messenger Telegram.

# General Introduction

The architecture of openHAB considers extensions. Those extensions are categorized where the categories are 
* Actions,
* Bindings,
* Misc,
* Persistence,
* Transformations,
* User Interfaces, and
* Voice.

Those names are used in the Paper UI, which is used for managing extensions. You will find them if you navigate to the Add-ons section.

In most cases those notification mechanisms are encapsulated in Action extensions. With respect to the notification types we are talking about in this post, this affects mail and Telegram notifications. Sending mobile push notifications using the openHAB app involves [openHAB Cloud](https://myopenhab.org/) and is therefore part of the openHAB Cloud extension. This one is part of the Misc category.

# Basic Setup of Items and Sitemap

As preparation for the main topic of sending notifications, we first need to set up necessary items and the sitemap. We will create an item file (in my case it's named `notifications.items`) containing three switch items, i.e. one for each type of notification. Enabling one of those switches should later trigger the respective notification.

```groovy
Switch  Send_via_App_Switch         "Send via App"      <switch>
Switch  Send_via_Mail_Switch        "Send via Mail"     <switch>
Switch  Send_via_Telegram_Switch    "Send via Telegram" <switch>
```

We will show those switches in the openHAB Basic UI. That's why we create a sitemap file (in my case it's named `notifications.sitemap`) with the content shown below.

```groovy
sitemap notifications label="Notifications" {
    Frame label="Notifications" {
        Switch item=Send_via_App_Switch
        Switch item=Send_via_Mail_Switch
        Switch item=Send_via_Telegram_Switch
    }   
}
```

Once those items and the sitemap are created, you can open the Basic UI. Please make sure to open the sitemap we recently created.

![Basic UI with Three Switches for Sending Notifications](/img/notifications_basicui.png){: .center-image }

# Send Notifications via openHAB App

The openHAB Cloud integration offers the possibility to send mobile push notifications to smartphones, which have the openHAB app installed. In general we need to take the following steps:

1. Add the [openHAB Cloud Connector][oh-cloud] in your openHAB environment.
1. Create an account on [myopenhab.org][my-oh].
1. Install and configure the mobile App on your [iOS][app-ios] or [Android][app-android] smartphone.

Those steps are described in more detail in the [openHAB docs][oh-cloud]. You are also welcome to read one of my previous [blog posts][cloud-post], which gives a simple but detailed explanation.

One this is done, we can add a rule for sending the mobile push notification. We need a rule similar to the one below in the rules file (in my case it's named `notifications.rules`).

```groovy
rule "Send Notification via App"
when
    Item Send_via_App_Switch changed
then
    if(Send_via_App_Switch.state == ON)
    {
        logInfo("notifications", "Sending notification via app.")
        sendNotification("andreas.hirsch@dancingbee.de", 
            "This is our notification sent via app.")

        Send_via_App_Switch.postUpdate(OFF)
    }
end
```

This rule checks, if the state of the switch changed. If this is the case, we check the new status. If the switch was turned on, a log message is written and the notification is sent via the `sendNotification` command. The first parameter is the mail address of the user. This mail address needs to be assigned to the openHAB Cloud account your smart home environment is connected to. The app on your smartphone has to sign in with this mail address. The second parameter is the message to show. After the notification was sent, the switch is turned off again.

If we trigger the switch in the Basic UI, the notification is sent to the app which displays it as push notification on the smartphone. The notification on my smartphone is shown in the image below.

![Mobile Notification on iOS](/img/notifications_app.jpeg){: .center-image .image-50vh }

# Send Notifications via Mail

For sending notifications as mail, we need to install the [Mail add-on][oh-mail]. You can e.g. do that using the Paper UI as shown in the image below. Just search for the add-on and click on install. There is also a [documentation][oh-mail] available.

![Installing the openHAB mail add-on](/img/notifications_addon_mail.png){: .center-image }

```properties
hostname=your.hostname
username=your_smtp_username
password=your_smtp_password
from=openHAB <your@mail.address>
ssl=true
```

After the add-on is installed, we can start using it for sending mails. We now add a rule for this type of notification in the rule file (in my case named `notifications.rules`).

```groovy

```

# Send Notifications via Telegram
Uses the Telegram Bot API

[my-oh]: https://myopenhab.org/
[oh-cloud]: https://www.openhab.org/addons/integrations/openhabcloud/
[cloud-post]: {{ site.baseurl }}{% post_url 2017-11-04-enable-remote-access-to-a-local-openhab-instance %}
[app-android]: https://play.google.com/store/apps/details?id=org.openhab.habdroid
[app-ios]: https://itunes.apple.com/de/app/openhab/id492054521?mt=8
[oh-mail]: https://www.openhab.org/addons/actions/mail/