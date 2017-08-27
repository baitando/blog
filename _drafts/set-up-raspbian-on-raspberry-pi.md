---
layout: post
title:  "Set up Raspbian on Raspberry Pi"
category: IT
tags: raspberry-pi
comments: true
---

This post describes how to set up a Raspberry Pi using the Rasbpian distribution. In general there is a useful [installation guide][raspi-guide] already available. Based on this guide, this post additionally shows the necessary steps to do the basic setup of the newly installed Raspbian distribution.

## Download the image
First of all, you need to download the official Raspbian Lite image provided [here][raspi-download]. Currently this is [Raspbian Strecht Lite][stretch-relnotes]. For getting access to the contained image file please unzip the downloaded archive to a local directory (e.g. by using [7-Zip][7-Zip]).

## Write the image to the SD card
Nowadays you only need to download and install a tool called Etcher for being able to write the zipped image to a SD card. Please download the tool from [the Etcher website][etcher] and install it. It is available for Mac and Linux as well as for Windows.

If you use Windows 10, it might be the case that a permission error like the one below is shown.

![Error shown when launching Etcher without administrative rights]({{ site.url }}/assets/etcher_error.png){: .center-image }

For solving this issue, you need to start Etcher with administrative privileges. Once started successfuly, you need to select the image archive downloaded before. Afterwards the target drive needs to be selected. After double checking those settings you can start the flashing process.

![Necessary settings for Etcher]({{ site.url }}/assets/etcher_settings.png){: .center-image }

You can close Etcher and eject the SD card as soon as the finish message is shown.

## Start the system and log in
After the system image was successfully written to the SD card, you need to place the SD card in your Raspberry Pi and start it. For doing the setup please connect a screen and a keyboard to it. As an alternative you can connect using a TTL to USB converter as described in a [previous post][post-ttl].

Once the system is ready you can log in using the default credentials, i.e. with the username `pi` and the password `raspberry`.

## Create a new sudo user

As a good practice you should create a new user for yourself. The new user should be part of the sudo group. After the user exists you should set the password and exit the session.

{% highlight console %}
pi@raspberrypi:~$ sudo useradd -m -s /bin/bash dummyusr --groups sudo
pi@raspberrypi:~$ sudo passwd dummyusr
pi@raspberrypi:~$ exit
{% endhighlight %}

The new user should replace the default one. For being able to remove the predefined one named `pi` the previously performed log out is needed. Open a new session under your new user and delete the old user.

{% highlight console %}
dummyusr@raspberrypi:~$ sudo userdel -r pi
{% endhighlight %}

## Update system settings

There are some more useful setting you should configure. The easiest way to do this is using the `raspi-config` tool.

{% highlight console %}
dummyusr@raspberrypi:~$ sudo raspi-config
{% endhighlight %}

I suggest to configure the following settings:

* Hostname
* Locale
* Timezone
* Keyboard layout
* Wife country

Afterwards a reboot is needed. After the system is back online, the settings will be active.

[raspi-download]: https://www.raspberrypi.org/downloads/raspbian/
[raspi-guide]: https://www.raspberrypi.org/documentation/installation/installing-images/README.md
[stretch-relnotes]: http://downloads.raspberrypi.org/raspbian/release_notes.txt
[7-Zip]: http://www.7-zip.org/
[etcher]: https://etcher.io/
[post-ttl]: {{ site.baseurl }}{% post_url 2017-08-26-connect-to-raspberry-pi-using-a-ttl-to-usb-converter %}
