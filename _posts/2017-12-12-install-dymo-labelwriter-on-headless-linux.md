---
layout: post
title:  "Install Dymo LabelWriter on headless Linux"
category: IT
tags: print linux
comments: true
excerpt: Dymo offers a driver for using their printers in a Linux environment. This post describes the setup of a Dymo LabelWriter 450 on Ubuntu 17.10 for printing via CUPS.
---

I am interested in programmatically printing labels generated in a Linux environment. For getting started, I bought a Dymo LabelWriter 450. The initial setup on my Windows laptop was easy and printing the first label using the Dymo Label software was no big deal.

The next step of installing the printer in a Linux environment was not that easy. To summarize it, you need to use the Common UNIX Printing System ([CUPS][cups-wiki]) and install the Dymo LabelWriter as a printer with the matching driver. This blog post describes the necessary steps in more detail. Maybe this helps someone, who faces the same problems.

I am referring to Ubuntu 17.10 64 Bit, but the steps described here should work for other Debian based distributions too. It is assumed, that the printer has power and is connected to the Linux machine via USB cable.

# Connection check

First of all, you should check, if the printer was recognized properly. Please execute the command shown below and check, if the result looks similar. If your printer is not listed, you need to check the USB connection.

{% highlight terminal %}
vagrant@ubuntu-artful:~$ sudo lsusb
Bus 001 Device 002: ID 0922:0020 Dymo-CoStar Corp. LabelWriter 450
Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
{% endhighlight %}

# Install the driver and  CUPS

The next step is installing the Linux driver. Luckily, there is a [precompiled package][package] available. Please install it using the command below.

{% highlight terminal %}
sudo apt-get update
sudo apt-get install cups cups-client printer-driver-dymo
{% endhighlight %}

# Download and install the printer definition

The printer setup requires an appropriate PostScript Printer Definition ([PPD][ppd]) file. Unfortunately, this seems not to be part of the installation package. For this reason, we need to download the [CUPS driver provided by Dymo][cups-driver]. This is currently version 1.4.0, which can be downloaded [here][cups-download]. Afterwards the archive needs to be extracted. The model file is part of it and should be copied to the default model folder of CUPS.

{% highlight terminal %}
wget http://download.dymo.com/dymo/Software/Download%20Drivers/Linux/Download/dymo-cups-drivers-1.4.0.tar.gz
tar -xzf dymo-cups-drivers-1.4.0.tar.gz
sudo mkdir -p /usr/share/cups/model
sudo cp /home/vagrant/dymo-cups-drivers-1.4.0.5/ppd/lw450.ppd /usr/share/cups/model/
{% endhighlight %}

# Add the printer

As final installation step we need to find out the address of the printer for being able to register it. The ``lpinfo`` (see [manpage][lpinfo-man]) command shows all available printers. There should be an entry referring to the Dymo printer.

{% highlight terminal %}
vagrant@ubuntu-artful:~$ sudo lpinfo -v
file cups-brf:/
network socket
network beh
network http
network lpd
network ipps
network https
network ipp
serial serial:/dev/ttyS0?baud=115200
direct usb://DYMO/LabelWriter%20450?serial=01010112345600
{% endhighlight %}

Once we know the printer address (here it is ``usb://DYMO/LabelWriter%20450?serial=01010112345600``), we can install it using ``lpadmin`` (see [manpage][lpadmin-man]) like shown below. The ``-p`` parameter specifies the display name of the printer, ``-v`` is used for the printer address and the ``-P`` parameter points to location of the printer definition file.

{% highlight terminal %}
lpadmin -p dymo -v usb://DYMO/LabelWriter%20450?serial=01010112345600 -P /usr/share/cups/model/lw450.ppd
{% endhighlight %}

Afterwards, the ``lpstat`` (see [manpage][lpstat-man]) command is used for listing all installed printers. The list should now contain the printer we previously installed.

{% highlight terminal %}
vagrant@ubuntu-artful:~$ lpstat -v
device for dymo: usb://DYMO/LabelWriter%20450?serial=01010112345600
{% endhighlight %}

Last but not least, we need to start the printer via ``cupsenable`` (see [manpage][cupsenable-man]) followed by the printer name we chose previously. Additionally, the printer is configured to accept jobs sent to it for printing. This is done by submitting the ``cupsaccept`` (see [manpage][cupsaccept-man]) command followed by the printer's name.

{% highlight terminal %}
sudo cupsenable dymo
sudo cupsaccept dymo
{% endhighlight %}

# Test the printer 

It makes sense to test the printer by printing some sample text. For this reason, we create a simple text file with some dummy text. This file is then sent to the printer for processing it by submitting the ``lp`` command (see [manpage][lp-man]). The ``-d`` parameter specifies the name of device to choose for printing and should match the one you chose while installing the printer. If you like to use the default printer, you can omit this parameter.

{% highlight terminal %}
echo Hello > test.txt
lp -d dymo test.txt
{% endhighlight %}

If everything works fine, your printer should now print a label containing the dummy text. This should look similar to the screenshot below.

![First label printed on a Dymo LabelWriter from a Linux environment]({{ site.url }}/assets/dymo_print_example_small.jpg){: .center-image }

# Set as default printer

If you like to configure your printer as the default one, you can use the ``lpoptions`` (see [manpage][lpoptions-man]) command as shown below. The ``-d`` parameter specifies the name of the new default printer. This is the name you chose while installing the printer.

{% highlight terminal %}
sudo lpoptions -d dymo
{% endhighlight %}

# Conclusion

Installing a Dymo LabelWriter on Windows (and probably Mac) using the Dymo Label software is easy. For reaching the same goal on headless Linux requires some more steps. You need to install an appropriate driver, provide the printer definition file and register the device as a printer. This is normal CUPS related configuration. If you are experienced in this area, the setup of the Dymo LabelWriter is no challenge. If not, you can use the instructions given by this blog post.  

[package]: https://packages.ubuntu.com/source/artful/dymo-cups-drivers
[sdk]: http://www.dymo.com/en-US/online-support-sdk
[cups-driver]: http://www.dymo.com/en-US/dymo-label-sdk-and-cups-drivers-for-linux-dymo-label-sdk-cups-linux-p--1
[cups-download]: http://download.dymo.com/dymo/Software/Download%20Drivers/Linux/Download/dymo-cups-drivers-1.4.0.tar.gz
[lpoptions-man]: https://www.cups.org/doc/man-lpoptions.html
[lp-man]: https://www.cups.org/doc/man-lp.html
[cupsenable-man]: https://www.cups.org/doc/man-cupsenable.html
[cupsaccept-man]: https://www.cups.org/doc/man-cupsaccept.html
[lpstat-man]: https://www.cups.org/doc/man-lpstat.html
[lpadmin-man]: https://www.cups.org/doc/man-lpadmin.html
[lpinfo-man]: https://www.cups.org/doc/man-lpinfo.html
[cups-wiki]: https://en.wikipedia.org/wiki/CUPS
[ppd]: https://www.cups.org/doc/spec-ppd.html