---
layout: post
title:  "openHAB Development With Intellij Idea"
category: IT
tags: openhab
bigimg: /img/bigimg_automation.jpg
credname: Daniele Levis Pelusi
credurl: https://unsplash.com/photos/Pp9qkEV_xPk
comments: true
excerpt: Thanks to the migration to BND Tools some time ago, it is now possible to develop openHAB extensions with Intellij Idea. I would like to show, how such a setup could look like.
---

I like openHAB, but the hurdles for contributing were too high for me in the past. After I switched from Eclipse to Intellij Idea several years ago, it felt cumbersome to develop OSGI bundles in Eclipse. Thanks to the switch to BND Tools, it is now easily possible to develop with Intellij Idea.


Currently I'm running two Synology DS218+ NAS systems.
One of them is located in my apartment, the other one runs in the house of my parents.
I connected both networks via VPN.
Therefore it is possible to access the applications running in the local network from both locations.

Due to the fact that I'm running a DNS server on each NAS with the configuration of the whole network, I always had to manually configure all DNS entries on both systems using the user interface provided by the DNS Server Synology package.

Unfortunately this ended in chaos.
After two years, the DNS configuration of both NAS systems differed.
Fixing this, I thought about ways to avoid the manual, redundant configuration process.
I ended up with a first attempt to automate things using scripts.

The blog post will explain what I found out and how my setup currently looks like.

# How the DNS Server Stores the Configuration

First of all I had a closer look at how the DNS Server Synology package basically works.
I found out, that the application and its configuration is located in the `/volume1/@appstore/DNSServer` directory on my NAS.
I guess that this only depends on the volume you use for the installation.
Besides that the path will most likely look similar.

Further analysis led to the conclusion, that the DNS zone configuration resides in the `/volume1/@appstore/DNSServer/named/etc/zone/master` directory.
This directory contains one file per configured DNS zone.
The file called `myzone.private` for the forward zone looks like shown below.

```
$ORIGIN myzone.private.
$TTL 86400
myzone.private. IN SOA ns.myzone.private. mail.myzone.private. (
	2019122211
	43200
	180
	1209600
	10800
)
myzone.private.              NS	ns.myzone.private.
ns.myzone.private.           A  192.168.0.1

mytest.myzone.private. 86400 A  192.168.0.1
```

The file called `0.168.192.in-addr.arpa` defines the reverse zone and looks like shown below.

```
$ORIGIN 0.168.192.in-addr.arpa.
$TTL 86400
0.168.192.in-addr.arpa. IN SOA ns.0.168.192.in-addr.arpa. mail.0.168.192.in-addr.arpa. (
	2019122211
	43200
	180
	1209600
	10800
)
0.168.192.in-addr.arpa.         NS  ns.myzone.private.

1.0.168.192.in-addr.arpa. 86400 PTR mytest.myzone.private.
```

The end of both configuration files looks pretty self-explaining and also the rest looks like it can be handled.

Later on I recognized, that there is one important part, which you maybe don't see immediately.
There are some static settings, but the string `2019122211` is a serial identifier.
As such, it needs to be increased with every changed configuration - otherwise changes are maybe not rolled out and the DNS Server package will show errors.

I did some experiments, changed some values and checked how the result looks like in the user interface of the DNS Server package.
Doing this, the question of how to reload the configuration came up.
After some more investigation, I found the script `/volume1/@appstore/DNSServer/script/reload.sh`.
If a change in the configuration file is done, it will become effective as soon as this reload script is executed.

# How to Reduce Manual Configuration Effort

Knowing this, I thought about a solution which reduces the manual configuration effort.
I came up with a solution, which looks as follows:

* The zone configuration files are versioned in a Git repository.
* Together with the configuration, a deployment script is provided in the Git repository.
* The Git repository is cloned to my laptop and the deployment runs from there.

The deployment script connects to one NAS after the other and automates the following steps:
* Create a backup of the current zone configuration located in `/volume1/@appstore/DNSServer/named/etc/zone/master`.
* Upload the new configuration from the laptop to the `/volume1/@appstore/DNSServer/named/etc/zone/master` directory on the NAS.
* Trigger a configuration reload of the DNS Server by executing `/volume1/@appstore/DNSServer/script/reload.sh`.

This procedure helps to reduce manual configuration effort, but it does not make it obsolete.
The automated part here is related to the ongoing maintenance and evolution of the DNS configuration.
The initial setup of the DNS Server is the part, which is still manual work.

I would really like to have a complete configuration as code including package installation, zone setup etc.
But compared to the effort required for this, I decided to first only realize the automation for ongoing topics.

## Conclusion
It was hard to put all the parts together to be able to deploy at least the zone configuration files of the DNS Server package.
But in the end it succeeded.

The solution already helps to minimize errors and to keep the configuration consistent across both NAS systems.
