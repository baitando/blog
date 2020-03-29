---
layout: post
title:  "Using DYMO LabelWriter 450 As Network Printer On Mac"
category: IT
tags: print network
bigimg: /img/bigimg_printer.jpg
credname: Wendelin Jacober
credurl: https://www.pexels.com/de-de/foto/arbeit-ausrustung-business-drinnen-1440504/
comments: true
excerpt: The DYMO LabelWriter 450 is not a itself capable to act as a network printer. But with a proper router, it is possible to expose the printer in the network. This post explains the setup with a FRITZ!Box and a Mac as user.
---

Some time ago I wrote a blog post which explained how to [use a DYMO LabelWriter 450 on a headless Linux][post-dymo-headless].
This time I would like to show, how the setup can be pimped to expose the labelling machine as a network printer.
For me this was an interesting topic, because the DYMO LabelWriter 450 is a pure USB printer.
Network capabilities are available with other DYMO products, but at a much higher price.

I decided to check, if there are cheaper ways to make my existing label writer available in the local network.
My setup consists of:

- DYMO LabelWriter 450
- AVM FRITZ!Box 6660 Cable, but also tested with AVM FRITZ!Box 6490
- An Apple MacBook

# Expose The Printer To The Network

The FRITZ!Box comes with the ability to act as a print sever. It is able to expose a USB connected printer in the local network.
The label writer is plugged in the USB port of the FRITZ!Box and must be turned on.

![DYMO LabelWriter 450 Connected To FRITZ!Box](/img/dymo_fritz_connected.png)

The label writer should automatically show up in the user interface of the router.
If you navigate to `Home Network` > `USB Devices` > `Overview` it should look similar to the screenshot above.

In fact this is everything which is necessary to make the DYMO LabelWriter 450 available as a network printer.

# Setup The Network Printer On Mac

As a first step I installed the DYMO software from the [website][dymo-website] to make sure, that the necessary drivers for the device are available.
To connect the printer, open the printer and scanner system settings to add a new printer.
Make sure to go to the `IP` section.

![Add DYMO LabelWriter 450 Connected As Network Printer On Mac](/img/dymo_fritz_add.png)

Use the IP or the name of the FRITZ!Box as address and make sure to select `HP Jetdirect - Socket` as the protocol.
Then select the driver of the the `DYMO LabelWriter 450`.

# Pitfalls

The things explained so far seem to be easy, but there were some pitfalls.
I think it is worth to talk about this to save others some headache.

The first problem in my setup was, that I previously used the USB port of the FRITZ!Box as a remote USB port.
With this setting and an additional piece of software installed, you can use the USB port of the FRITZ!Box on a Windows computer like it would be one of the local machine.
Unfortunately this setting prevents that the USB connected printer is exposed as network printer.
So make sure that this setting is deactivated like shown in the screenshot below.

![Deactivated Remote USB Port on FRITZ!Box](/img/dymo_fritz_remote_port.png)

The second pitfall was, that I was not aware that it is necessary to use the `HP Jetdirect - Socket` entry as protocol.
Doing some research I found some describtions in the web which stated explicitly this and I can confirm that it works.

The third topic was, that I did not install the DYMO software in advance.
As a consequence, there was no suitable driver available.
Installing the complete DYMO software installs it and makes it possible to select the `DYMO LabelWriter 450` entry.

![Deactivated Remote USB Port on FRITZ!Box](/img/dymo_fritz_software.png)

In addition, you can of course use the DYMO software to create and print labels with the vendor provided software.

# Conclusion

This post explained the necessary steps to expose a DYMO LabelWriter 450 with no own network capabilities as a network printer by using the FRITZ!Box as a print server.
In addition it was shown, how this remote device can be connected as a network printer to use it on a Mac.

There are some pitfalls - but if you master them, you can save some money compared to buying a network capable label writer.

[post-dymo-headless]: {% post_url 2017-12-12-install-dymo-labelwriter-on-headless-linux %}
[dymo-website]: https://www.dymo.com/de-DE/labelwriter-450-label-printer