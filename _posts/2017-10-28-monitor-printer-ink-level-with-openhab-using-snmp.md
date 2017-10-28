---
layout: post
title:  "Monitor printer ink levels with openHAB using SNMP"
category: IT
tags: smart-home openhab
comments: true
excerpt: Monitoring the ink levels of printers with openHAB is possible using the existing SNMP binding. This post shows an example configuration.
---

A short time ago I decided to monitor the ink levels of my HP Officejet 6700 printer in openHAB. The main goals were:

* Show the current ink level of each cartridge in the Basic UI.
* Automatically send a notification as soon as a cartridge is nearly empty.

This post describes the necessary configuration steps for the specific printer type. This example is based on [openHAB 2][openhab] and uses the existing [legacy SNMP binding][snmp-binding]. Please have a look at [Wikipedia][snmp-wikipedia] for a description of SNMP.

A good starting point is [this blog post][ioblogger-post]. It shows, how the ink levels can be retrieved via SNMP. Based on this description, I started implementing the retrieval of the printer data via SNMP inside openHAB.

## Prerequisites and SNMP connection check

Using a static IP for the printer simplifies the configuration. Please ensure, that the printer is set up accordingly.

Once the IP setup of the printer is done, it makes sense to check the SNMP connection. It should be available by default - at least in my case it was already available.

On Linux (Raspbian in this example) you can check this by installing the ``snmp`` package and issuing the ``snmpwalk`` command.

{% highlight terminal %}
dummyusr@raspberrypi:~$ sudo apt-get update
dummyusr@raspberrypi:~$ sudo apt-get install snmp
dummyusr@raspberrypi:~$ sudo apt-get update
dummyusr@raspberrypi:~$ snmpwalk -v1 -c public 192.168.0.123
iso.3.6.1.2.1.1.1.0 = STRING: "HP ETHERNET MULTI-ENVIRONMENT"
...
iso.3.6.1.2.1.43.11.1.1.6.0.1 = STRING: "black ink"
iso.3.6.1.2.1.43.11.1.1.6.0.2 = STRING: "yellow ink"
iso.3.6.1.2.1.43.11.1.1.6.0.3 = STRING: "cyan ink"
iso.3.6.1.2.1.43.11.1.1.6.0.4 = STRING: "magenta ink"
iso.3.6.1.2.1.43.11.1.1.7.0.1 = INTEGER: 15
iso.3.6.1.2.1.43.11.1.1.7.0.2 = INTEGER: 15
iso.3.6.1.2.1.43.11.1.1.7.0.3 = INTEGER: 15
iso.3.6.1.2.1.43.11.1.1.7.0.4 = INTEGER: 15
iso.3.6.1.2.1.43.11.1.1.8.0.1 = INTEGER: 297
iso.3.6.1.2.1.43.11.1.1.8.0.2 = INTEGER: 114
iso.3.6.1.2.1.43.11.1.1.8.0.3 = INTEGER: 117
iso.3.6.1.2.1.43.11.1.1.8.0.4 = INTEGER: 125
iso.3.6.1.2.1.43.11.1.1.9.0.1 = INTEGER: 238
iso.3.6.1.2.1.43.11.1.1.9.0.2 = INTEGER: 68
iso.3.6.1.2.1.43.11.1.1.9.0.3 = INTEGER: 94
iso.3.6.1.2.1.43.11.1.1.9.0.4 = INTEGER: 54
...
{% endhighlight %}

The output should be rather long and the first line should look similar to the one listed above. Not all the lines are listed there.

## Identify relevant data

Relevant data for the ink level use case is provided according to [RFC 3805][rfc-3805]. This RFC defines models and manageable objects for printing environments. For a better understanding of the numeric OIDs I recommend [this overview][printer-mib]. The following OIDs are relevant for our use case:

| OID                         | Description                                       |
| ----------------------------|---------------------------------------------------|
| 1.3.6.1.2.1.43.11.1.1.6.0.X | Color                                             |
| 1.3.6.1.2.1.43.11.1.1.7.0.X | Measurement unit (15 means tenths of milliliters) |
| 1.3.6.1.2.1.43.11.1.1.8.0.X | Maximum capacity in unit given before             |
| 1.3.6.1.2.1.43.11.1.1.9.0.X | Current capacity in unit specified before         | 

The X contained in the OIDs listed above needs to be replaced by

* 1 for the black cartridge,
* 2 for the yellow cartridge,
* 3 for the cyan cartridge,
* 4 for the magenta cartridge. 

Using those OIDs we can do another check. The command below retrieves the color name of the first cartridge.

{% highlight console %}
dummyusr@raspberrypi:~$ snmpget -v1 -c public 192.168.0.123 1.3.6.1.2.1.43.11.1.1.6.0.1
iso.3.6.1.2.1.43.11.1.1.6.0.1 = STRING: "black ink"
{% endhighlight %}

If the result looks similar to the one shown above, everything works as desired.

## Retrieve data with openHAB 2 by using the SNMP binding

There is a [(legacy) SNMP binding][snmp-binding] available for openHAB 2. We will use it for retrieving the data from the printer. For this reason, the first step is installing the binding.

![Installing the SNMP binding via Paper UI]({{ site.url }}/assets/oh_snmp_install_binding.png){: .center-image }

This can either be done using the Paper UI (see screenshot above) or by adding it to the ``services/addons.cfg`` file of your installation (see below).

{% highlight console %}
# A comma-separated list of bindings to install (e.g. "sonos,knx,zwave")
binding = snmp1
{% endhighlight %}

If your openHAB installation runs on a Unix machine, you need to do additional configuration work. This is caused by the fact, that the port chosen by the binding can only be bind by a privileged user - but openHAB should not run under such a user. The [binding documentation][snmp-binding] gives further explanation. I'm not really sure, if the workaround mentioned there is up to date. In my case it was enough to change the port configured in ``services/snmp.conf`` to a non-privileged one.

{% highlight console %}
# Listening Port (optional, defaults to '162')
port=12345
{% endhighlight %}

Once the binding is installed, we can proceed with the item configuration (here I use the file ``items/test.items``). An example for the black cartridge is shown below. The item configuration for the three remaining cartridges should be similar to this. For further information regarding the syntax etc. please have a look at the [binding documentation][snmp-binding].

{% highlight console %}
Number Max_Black "Max. Black [%.0f]"    <colorwheel> { snmp="<[192.168.0.123:public:.1.3.6.1.2.1.43.11.1.1.8.0.1:10000]" }
Number Cur_Black "Cur. Black [%.0f]"    <colorwheel> { snmp="<[192.168.0.123:public:.1.3.6.1.2.1.43.11.1.1.9.0.1:10000]" }
Number Lvl_Black "Lvl. Black [%.0f %%]"	<colorwheel>		
{% endhighlight %}

The example shows, that only the maximum and the current capacity are retrieved via SNMP. The capacity level needs to be calculated. As far as I know, there is no possibility to specify this calculation in the item configuration. The calculation is therefore defined as a rule in ``rules/test.rules``.

As soon as the maximum or the current capacity of the black cartridge changed, the relative capacity is calculated and assigned to the corresponding item. Similar rules need to be defined for the other cartridges also.

{% highlight console %}
rule "Calculate ink level of black cartridge"
when
  Item Max_Black changed or
  Item Cur_Black changed
then
  if (Max_Black.state instanceof DecimalType && Cur_Black.state instanceof DecimalType) {
    Lvl_Black.postUpdate((Cur_Black.state as DecimalType) / (Max_Black.state as DecimalType) * 100)
  } else {
    Lvl_Black.postUpdate(UNDEF)
  }
end
{% endhighlight %}

Afterwards we create a sitemap for showing the data. The dummy one containing the data of the black cartridge is located in the file ``sitemaps/test.sitemap``.

{% highlight console %}
sitemap test label="Test"
{
	Frame label="Test" {
		Text item=Cur_Black
		Text item=Max_Black
		Text item=Lvl_Black		
	}   		
}
{% endhighlight %}

Now we can check the result using the Basic UI. Please navigate to it and check, if the result looks similar to the one shown in the screenshot below.

![Ink data of black cartridge shown in Basic UI]({{ site.url }}/assets/oh_ink_basic_ui.png){: .center-image }

# Send notification if cartridges are nearly empty

The next step is monitoring the ink levels and sending a notification, if a cartridge is nearly empty. One of the most common ways of notifying is probably sending a mail.

![Installing the mail action via Paper UI]({{ site.url }}/assets/oh_mail_install_action.png){: .center-image }

For being able to send a mail from within openHAB you need to add the mail action via Paper UI or configuration in your ``services/addons.cfg`` file. Necessary configuration steps are described as part of the [action's documentation][action-mail].

{% highlight console %}
# A comma-separated list of actions to install (e.g. "mail,pushover")
action = mail
{% endhighlight %}

Once the mail action is available, the rule for checking the ink levels and sending the notification can be specified. The example below checks, if the ink level of the black cartridge is less than or equal 20%. If this is true, an appropriate mail notification is send. Similar rules need to be added for all other cartridges.

{% highlight console %}
rule "Warn if black cartridge is nearly empty"
when
	Item Lvl_Black changed
then
	if(Lvl_Black.state instanceof DecimalType && Lvl_Black.state <= 20)
	{
		sendMail("ando.hirsch@gmail.com", "Black cartridge nearly empty", "The black cartridge of the printer is nearly empty. Please consider buying a new one.")
	}
end
{% endhighlight %}

This was the last step necessary for showing the printer cartridge ink levels in openHAB. The rules specified last will notify us, if any of the cartridges are nearly empty.

[ioblogger-post]: https://www.ioblogger.de/2016/09/hp-officejet-6700-tintenstand-munin/
[openhab]: https://www.openhab.org
[snmp-binding]: http://docs.openhab.org/addons/bindings/snmp1/readme.html
[snmp-wikipedia]: https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol
[rfc-3805]: http://www.networksorcery.com/enp/rfc/rfc3805.txt
[printer-mib]: http://www.oidview.com/mibs/0/Printer-MIB.html
[action-mail]: http://docs.openhab.org/addons/actions/mail/readme.html