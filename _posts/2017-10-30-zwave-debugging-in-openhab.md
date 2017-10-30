---
layout: post
title:  "Z-Wave debugging in openHAB"
category: IT
tags: smart-home openhab
comments: true
excerpt: The inclusion of new Z-Wave devices in an openHAB environment is sometimes easy - and sometimes not. Logging mechanisms and the Z-Wave log file viewer are helpful debugging tools.
---

I had some problems including a new Z-Wave flood sensor in my openHAB managed smart home environment. While analyzing this, I found out, that there is a very useful log file viewer provided by [the author][jackson-homepage] of the [Z-Wave binding][zwave-binding].

This blog post assumes, that you have an up and running openHAB environment including the Z-Wave binding and one or more Z-Wave devices available. I'll describe how to configure the logging and how to open your logs in the log file viewer.

# Configure the log level

The configuration of the logging mechanisms is described in the [openHAB documentation][oh-logging]. Normally I do the configuration using the Karaf console. You need to open a terminal on the machine hosting your openHAB instance. Then connect to the Karaf console via ``ssh -p 8101 openhab@localhost ``. By default, the password is ``habopen``. This password, the port (here it is ``8101``) and the username (here it is ``openhab``) might need to be adjusted according to your setup.

{% highlight terminal %}
  ____  ____  ___  ____  / / / /   |  / __ )
 / __ \/ __ \/ _ \/ __ \/ /_/ / /| | / __  |
/ /_/ / /_/ /  __/ / / / __  / ___ |/ /_/ /
\____/ .___/\___/_/ /_/_/ /_/_/  |_/_____/
    /_/                        2.1.0
                               - release build -

Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown openHAB.
{% endhighlight %}

Once you are connected, the terminal shows some output similar to the one above. Afterwards, you can set the log level of the Z-Wave binding to debug.

{% highlight terminal %}
openhab> log:set debug org.openhab.binding.zwave
{% endhighlight %}

Please keep in mind, that those changes are lost upon restart. If you'd like to make persistent changes, you can set the log level in a configuration file (located in ``/var/lib/openhab2/etc/org.ops4j.pax.logging.cfg`` if you did the setup using the package e.g. provided on Raspbian). For setting the Z-Wave binding log level to debug, you should add a line similar to the one shown below or change the existing one if present.

{% highlight terminal %}
log4j.logger.org.openhab.binding.zwave = DEBUG
{% endhighlight %}

From this time on, more Z-Wave related log messages are written to the log file. It is e.g. located in ``/var/log/openhab2/openhab.log`` if your instance is hosted on Raspbian and if you used the available installation package.

# Analyze the log file with the log file viewer

The log file produced by openHAB can be analyzed with a [log file viewer][log-viewer]. It is provided [online][log-viewer] by the author of the Z-Wave binding.

First of all you need to select the log file, which could e.g. be located in ``/var/log/openhab2/openhab.log``. According to the notification shown there, all processing is done on your local machine. There seems to be no upload of your local file unless specifically stated.

![Choose the log file to analyze]({{ site.url }}/assets/zwave-logviewer_choose-file.png){: .center-image }

Depending on the size of your log file, processing might take some seconds. Once this finished, you can jump to the log view. It shows Z-Wave related messages contained in your log file in a structured way. From my point of view, this is much more readable and understandable than reading the log messages in a text editor. 

![Z-Wave related log messages are visualized in the log viewer]({{ site.url }}/assets/zwave-logviewer_analyze.png){: .center-image }

An additional benefit is the possibility to filter the messages by node. This was helpful in my case, because my device was already included in openHAB. Therefore I knew the node ID (shown in Paper UI) and was able to filter the log messages accordingly.  

![Z-Wave related log messages can be filtered by node ID]({{ site.url }}/assets/zwave-logviewer_filter.png){: .center-image }

Last but not least, there is some more data related to each node provided in a separate tab.

![Some additional node related data is provided]({{ site.url }}/assets/zwave-logviewer_node-info.png){: .center-image }

# Conclusion
Besides the possibilty to analyze Z-Wave messages contained in the openHAB log file, the online [log file viewer][log-viewer] is a great tool for analyzing Z-Wave related problems.

[jackson-homepage]: http://www.cd-jackson.com
[zwave-binding]: http://docs.openhab.org/addons/bindings/zwave/readme.html
[oh-logging]: http://docs.openhab.org/administration/logging.html
[log-viewer]: http://www.cd-jackson.com/index.php/openhab/zwave-log-viewer