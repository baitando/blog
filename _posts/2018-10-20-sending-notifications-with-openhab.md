---
layout: post
title:  "Sending Notifications with openHAB"
category: IT
tags: openhab smart-home
bigimg: /img/bigimg_notification.jpeg
credname: Snapwire
credurl: https://www.pexels.com/photo/beach-bottle-cold-daylight-292426/
comments: true
excerpt: There are several types of notifications supported by openHAB. This post shows how to send notifications via mail, via Telegram chat and as mobile push notification.
---

There are [openHAB][oh-homepage] add-ons available for sending different types of notifications from within your openHAB based smart home environment. I will pick three of them and show the basic configuration, which can be adapted to your own needs. We will talk about the following notification types:

* Notifications sent as mobile push notification.
* Notifications sent as mail.
* Notifications sent to a Telegram chat.

We will define one switch item per notification type and show them in the Basic UI. Once we trigger one of these switches, a notification type specific rule will run and send the notification. After the notification is sent, the switch will change back to the off state. 

We start with the basic setup of necessary items and the sitemap for the Basic UI. After that we continue with a closer look on the necessary configuration for each notification type.

# Prerequisites

This blog post requires a running openHAB instance and basic knowledge regarding add-on installation and openHAB configuration. All explanations are based on openHAB 2.3 but will most likely also work with different versions.

If you don't have an openHAB instance yet, you can have a look at one of my previous blog posts, which describes the [installation of openHAB on a Raspberry Pi][setup-post]. In case you are not yet familiar with the add-on installation process, there is a good documentation available on the [openHAB website][oh-installation]. We will use the Paper UI based approach.

All file system paths I mention are valid for an APT based installation. This is e.g. the case, if your instance runs on Raspbian and you used the available installation package. For further details please have a look at the [documentation][oh-paths].

# Basic Setup of Items and Sitemap

We first need to create all necessary items. They are defined in an item file (in my case it's `/etc/openhab2/items/notifications.items`). We need one switch item per notification type. Enabling one of the switches will later trigger a rule for sending the notification.

```groovy
Switch  Send_via_App_Switch         "Send via App"      <switch>
Switch  Send_via_Mail_Switch        "Send via Mail"     <switch>
Switch  Send_via_Telegram_Switch    "Send via Telegram" <switch>
```

Those switches should be shown in the Basic UI. That's why we create a sitemap file (in my case `/etc/openhab2/sitemaps/notifications.sitemap`) with the content shown below.

```groovy
sitemap notifications label="Notifications" {
    Frame label="Notifications" {
        Switch item=Send_via_App_Switch
        Switch item=Send_via_Mail_Switch
        Switch item=Send_via_Telegram_Switch
    }   
}
```

Once those items and the sitemap are created, you can open the Basic UI to check the result. Please make sure to use the sitemap we recently created. The page should look similar to the screenshot below.

![Basic UI with Three Switches for Sending Notifications](/img/notifications_basicui.png){: .center-image }

# Send Mobile Push Notifications

We will have a closer look at sending mobile push notifications with [openHAB Cloud](https://myopenhab.org/). It offers the possibility to send mobile push notifications to a smartphone, which has the openHAB app installed and is connected to an openHAB Cloud account. The proper setup of an openHAB Cloud connection consists of the following steps:

1. Install the [openHAB Cloud Connector][oh-cloud].
1. Create an account on [myopenhab.org][my-oh] if you don't have one yet.
1. Install the [iOS][app-ios] or [Android][app-android] app on your smartphone.
1. Connect the app to your openHAB Cloud account.

Those steps are described in the [openHAB docs][oh-cloud]. You are also welcome to read one of my previous blog posts, which describes [this in more detail][cloud-post].

We can now add a rule for sending the mobile push notification once the switch in the Basic UI is triggered. We need a rule similar to the one below in a rule file (in my case `/etc/openhab2/rules/notifications.rules`).

```groovy
rule "Send Mobile Push Notification"
when
    Item Send_via_App_Switch changed
then
    if(Send_via_App_Switch.state == ON)
    {
        logInfo("notifications", "Sending notification via app.")
        sendNotification("<recipient_mail_address>", 
            "This is our notification sent via app.")

        Send_via_App_Switch.postUpdate(OFF)
    }
end
```

This rule checks, if the state of the switch changed. If the status changed to on, a log message is written and the notification is sent via the `sendNotification` command, which takes two parameters. After the notification was sent, the switch is turned off again.

1. The first parameter is the mail address of the recipient user. It must be assigned to the openHAB Cloud account your smart home environment is connected to. Of course this needs to be the same mail address you configured in the smartphone app.
1. The second parameter is the message to send.

If we now trigger the switch in the Basic UI, the notification is sent to the app which displays it as push notification on the smartphone. The notification on my smartphone is shown in the image below.

![Mobile Notification on iOS](/img/notifications_app.jpeg){: .center-image .image-50vh }

# Send Mail Notifications

For sending mail notifications, we need to install the [Mail add-on][oh-mail]. I did this by using the the Paper UI like shown in the screenshot below. Just search for the add-on and click on install. There is also a [documentation][oh-mail] available.

![Installing the openHAB mail add-on](/img/notifications_addon_mail.png){: .center-image }

After the installation finished, you will find a new file named `mail.cfg`in the `/etc/openhab2/services` directory. The default file contains many comments which show how to configure the add-on. Below you will find the most relevant settings. They define the SMTP account used for sending the mails. Please make sure to set your specific values.

```properties
hostname=<your_smtp_hostname>
username=<your_smtp_username>
password=<your_smtp_password>
from=openHAB System <<sender_mail_address>>
ssl=true
```

With this configuration done we can now add a new rule in the rule file (in my case `/etc/openhab2/rules/notifications.rules`). The most relevant part is the `sendMail` command, which has some parameters. The rest is similar to the app notification rule.

1. The first parameter defines the mail address of the recipient. 
1. The second one defines the subject.
1. The third one defines the text to send.

The rule is shown below. Please make sure to adjust the parameters according to your needs. There are more features available, which are explained in the [documentation][oh-mail]. You can e.g. add one or more attachments. 

```groovy
rule "Send Notification via Mail"
when
    Item Send_via_Mail_Switch changed
then
    if(Send_via_Mail_Switch.state == ON)
    {
        logInfo("notifications", "Sending notification via mail.")
        sendMail("<recpient_mail_address>", "Mail Notification", 
            "This is our notification sent via mail.")

        Send_via_Mail_Switch.postUpdate(OFF)
    }
end
```

If we now trigger the switch in the Basic UI, a mail is sent. The screenshot below shows the example sent to my mail address.

![Mail Notification](/img/notifications_mail.png){: .center-image }

# Send Notifications to a Telegram Chat

Finally we will have a closer look at sending notifications to a [Telegram][telegram] chat. We will use the [openHAB Telegram action][telegram-action]. For further details not covered in this blog post, you can have a look at the [documentation][telegram-action] of this add-on. 

First of all you need to create a Telegram account if you don't already have one. Please follow these steps:

1. Download the iOS app, the Android app or open the Web app. Necessary links are listed on the [Telegram website][telegram-apps].
1. Enter the phone number of your smartphone.
1. You will receive a code on your smartphone with the phone number you entered before.
1. Type in the code to complete the registration.

The notification mechanism in openHAB uses the [Telegram Bot API][telegram-api]. We will now create a new bot, which will be used to send the notifications. 

1. Open a new chat with `BotFather` in Telegram. You can do that by typing the name in the search field and selecting the entry.
1. Send the message `/newbot`.
1. Give your new bot a name. I used `openHAB`. This is used for display purposes.
1. Enter the username of the bot. It must be unique on Telegram and must end with `bot`. I used `openHAB_baitando_bot`. The `BotFather` will tell you if the name you entered is already in use. In this case you will have to choose a different one.

If this was successful, the token for accessing the HTTP API is shown. We will need this token later, so please write it down. The complete chat history is shown in the image below. The token is displayed where I blurred the red text.

![Mail Notification](/img/notifications_telegram_botfather.png){: .center-image }

The next step is to get the chat ID. Please follow the steps listed below.

1. Open a new chat with your new chat bot.
1. Send a message in this chat. I simply sent the message `Hello`.
1. Open the URL `https://api.telegram.org/bot<token>/getUpdates` in your browser - but don't forget to replace `<token>`with the token you generated before.
1. Your browser will show a JSON result. Please write down the value of `result[0].message.chat.id`.

In my case the JSON content below is shown. The value we will need later is `474141047` in my case.

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

The installation process automatically creates the `telegram.cfg` file in the `/etc/openhab2/services` directory. Please replace `<chat_id>` and `<token>` with the values you have written down.

```properties
bots=openHAB

openHAB.chatId=<chat_id>
openHAB.token=<token>
```
Finally we add a new rule (in my case in `/etc/openhab2/rules/notifications.rules`), which sends the Telegram notification once we trigger the switch in the Basic UI. The most important command is `sendTelegram`. It is called with two parameters.

1. The first one needs to be the same value than used in the bot definition in `telegram.cfg`.
1. The second parameter is the message itself.

The rule is shown below. Please make sure to adjust the parameters according to your needs. There are more features available, which are explained in the [documentation][telegram-action]. You can e.g. add images to your notification.

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

There are several types of notifications you can send from within your openHAB based smart home environment. This blog post described three of those notifications types. Feel free to adapt the samples shown here to your own needs. You can also refer to the openHAB documentation of the add-ons, if you want to enhance your notifications.

Besides the notifications types mentioned here, openHAB provides much more extensions you can use. You can search for them e.g. in the [documentation][oh-addons] or in the add-ons section of the Paper UI.

[oh-homepage]: https://www.openhab.org
[oh-paths]: http://docs.openhab.org/installation/linux.html#file-locations
[my-oh]: https://myopenhab.org/
[oh-cloud]: https://www.openhab.org/addons/integrations/openhabcloud/
[cloud-post]: {{ site.baseurl }}{% post_url 2017-11-04-enable-remote-access-to-a-local-openhab-instance %}
[setup-post]: {{ site.baseurl }}{% post_url 2017-09-03-set-up-raspbian-on-raspberry-pi %}
[app-android]: https://play.google.com/store/apps/details?id=org.openhab.habdroid
[app-ios]: https://itunes.apple.com/de/app/openhab/id492054521?mt=8
[oh-mail]: https://www.openhab.org/addons/actions/mail/
[telegram]: https://telegram.org
[telegram-apps]: https://telegram.org/apps
[telegram-action]: https://www.openhab.org/addons/actions/telegram
[oh-addons]: https://www.openhab.org/addons/
[oh-installation]: https://www.openhab.org/docs/configuration/addons.html
[telegram-api]: https://core.telegram.org/bots/api