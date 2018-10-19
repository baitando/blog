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

After the installation finished, you will find a new file named `mail.cfg`in the `conf/services` directory. The default file contains many comments which show how to configure the add-on. Below you will find the most relevant settings. They specify the SMTP account used for sending the mails. Please make sure to set your specific values.

```properties
hostname=your.hostname
username=your_smtp_username
password=your_smtp_password
from=openHAB System <your@mail.address>
ssl=true
```

With this installation and configuration work done, we can start using the add-on for sending mails. We now add a rule for this type of notification in the rule file (in my case named `notifications.rules`). The most relevant part is the `sendMail` command, which has some parameters. The first one is the mail address of the recepient. The second one is the topic and the third one the text to send. The rest is similar to the app notification rule.

```groovy
rule "Send Notification via Mail"
when
    Item Send_via_Mail_Switch changed
then
    if(Send_via_Mail_Switch.state == ON)
    {
        logInfo("notifications", "Sending notification via mail.")
        sendMail("ando.hirsch@gmail.com", "Mail Notification", 
            "This is our notification sent via mail.")

        Send_via_Mail_Switch.postUpdate(OFF)
    }
end
```

If we now trigger the switch in the Basic UI, a mail is sent. The screenshot below shows an example that was sent to my mail address.

![Mail Notification](/img/notifications_mail.png){: .center-image }

# Send Notifications via Telegram

Finally we now will have a closer look at [Telegram][telegram] notifications. This is a pretty interesting notification type. We will use the [openHAB Telegram action][telegram-action], which comes with a [documentation][telegram-action] which shows the necessary steps. We will follow these steps now. 

First of all you need to create a Telegram account if you don't already have one.

1. Download the iOS app, the Android app or open the Web app. The links to these apps and some more are listed on the [Telegram website][telegram-apps].
1. Enter your phone number.
1. You will receive a code on your smartphone with the phone number you entered before.
1. Type in the code.

The notification mechanism in openHAB will leverage the Telegram Bot API. We will now create a new bot, which will be used to send the notifications via Telegram. 

1. Open a new chat with `BotFather` in Telegram.
1. Send the message `/newbot`.
1. Give your new bot a name. I used `openHAB`.
1. Enter the username of the bot. It must be unique on Telegram and must end with `bot`. I used `openHAB_baitando_bot`. The `BotFather` will tell you if the name you entered is already in use. In this case you would have to choose a different one.

If this was successful, the token for accessing the HTTP API is shown. We will need this token later. The complete chat history is shown in the image below. The token is displayed where I blurred the text in this screenshot.

![Mail Notification](/img/notifications_telegram_botfather.png){: .center-image }

The next step is to get the chat ID. Please follow the steps listed below.

1. Open a new chat with your new chat bot.
1. Send a message in this chat. I simply sent the message `Hello`.
1. Open the URL `https://api.telegram.org/bot<token>/getUpdates` in your browser - but don't forget to replace `<token>`with the token you generated before.
1. Your browser will show a JSON result. Please write down the value of `result[0].message.chat.id`.

In my case the JSON below is shown. The value we will need later is `474141047` in my case.

```javascript
{  
   "ok":true,
   "result":[  
      {  
         "update_id":494609500,
         "message":{  
            "message_id":1,
            "from":{  
               "id":474141047,
               "is_bot":false,
               "first_name":"Andreas",
               "last_name":"Hirsch",
               "language_code":"de-DE"
            },
            "chat":{  
               "id":474141047,
               "first_name":"Andreas",
               "last_name":"Hirsch",
               "type":"private"
            },
            "date":1539987708,
            "text":"Hello"
         }
      }
   ]
}
```

Now we have all necessary values for the configuration. Let's install the Telegram add-on. You can do this using the Paper UI like before. Just click on install and the add-on will be installed automatically. A screenshot is provided below.

![Installing the openHAB mail add-on](/img/notifications_addon_telegram.png){: .center-image }

The installation process automatically creates the `telegram.cfg` file in the `conf/services` directory. Please replace `<chat_id>` and `<token>` according to your setup.

```properties
bots=openHAB

openHAB.chatId=<chat_id>
openHAB.token=<token>
```
Finally we add a new rule, which is responsible of sending the Telegram notification once we trigger the switch in the Basic UI. The sample rule is shown below. The most important command is `sendTelegram`. It is called with two parameters. The first one needs to be the same value than used in the bot definition in `telegram.cfg`. The second parameter is the message itself.

```groovy
rule "Send Notification via Mail"
when
    Item Send_via_Telegram_Switch changed
then
    if(Send_via_Telegram_Switch.state == ON)
    {
        logInfo("notifications", "Sending notification via Telegram.")
        sendTelegram("openHAB", 
            "This is our notification sent via Telegram.")

        Send_via_Telegram_Switch.postUpdate(OFF)
    }
end
```

This rule will send the Telegram message once we trigger the switch in the Basic UI. The screenshot below shows the sample message sent to my Telegram chat.

![Telegram Notification](/img/notifications_telegram.png){: .center-image }

# Conclusion

There are several types of notifications you can send from within your openHAB based smart home environment. This blog post described three of those notifications types. Feel free to adapt the samples shown here to your own needs. You can also refer to the openHAB documentation of the add-ons used here.

Besides the notifications types mentioned here, openHAB provides much more extensions you can use. You can search for them e.g. in the [documentation][oh-addons] or in the add-ons section of the Paper UI.

[my-oh]: https://myopenhab.org/
[oh-cloud]: https://www.openhab.org/addons/integrations/openhabcloud/
[cloud-post]: {{ site.baseurl }}{% post_url 2017-11-04-enable-remote-access-to-a-local-openhab-instance %}
[app-android]: https://play.google.com/store/apps/details?id=org.openhab.habdroid
[app-ios]: https://itunes.apple.com/de/app/openhab/id492054521?mt=8
[oh-mail]: https://www.openhab.org/addons/actions/mail/
[telegram]: https://telegram.org
[telegram-apps]: https://telegram.org/apps
[telegram-action]: https://www.openhab.org/addons/actions/telegram
[oh-addons]: https://www.openhab.org/addons/