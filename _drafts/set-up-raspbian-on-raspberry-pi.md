---
layout: post
title:  "Set up Raspbian on Raspberry Pi"
category: IT
tags: raspberry-pi administration
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

## Start the system
After the system image was successfully written to the SD card, you need to place the SD card in your Raspberry Pi and start it.

[raspi-download]: https://www.raspberrypi.org/downloads/raspbian/
[raspi-guide]: https://www.raspberrypi.org/documentation/installation/installing-images/README.md
[stretch-relnotes]: http://downloads.raspberrypi.org/raspbian/release_notes.txt
[7-Zip]: http://www.7-zip.org/
[etcher]: https://etcher.io/
