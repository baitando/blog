---
layout: post
title:  "Integration of Homematic thermostats with openHAB"
category: IT
tags: smart-home openhab
comments: true
excerpt: My first smart home experience was setting up a Homematic gateway with some thermostats in my parents house. This post shows how to integrate such devices with openHAB.
---

One major strength of [openHAB][oh-homepage] is being able to integrate devices of different vendors using different technologies and communication protocols. This was my reason for switching to openHAB two years ago. Due to the fact, that my parents used a homogeneous Homematic environment with a gateway and some thermostats before, my first experience with openHAB was integrating those devices with the new openHAB smart home environment.

This post gives some hints and shows a working configuration. The setup consists of the following components:

- Raspberry Pi 3 running the Raspbian distribution
- Via APT package installed openHAB in version 2.1
- Homematic CCU2 gateway
- Homematic HM-CC-RT-DN thermostat

This blog post assumes, that the thermostat and the gateway are already connected and set up. Using the configuration described here will make the Homematic environment accessible for openHAB.

All file system paths are valid for an APT based installation. This is e.g. the case, if your instance is hosted on Raspbian and you used the available installation package. For further details please have a look at the [documentation][oh-paths].

# Connect openHAB and the Homematic CCU2 gateway

There is a [Homematic binding][oh-homematic-binding] available for openHAB 2, which is able to communicate with a Homematic CCU2 gateway using your network. 

![Installing the Homematic binding via Paper UI]({{ site.url }}/assets/oh_homematic_install_binding.png){: .center-image }

You can install the binding using the Paper UI as shown in the screenshot above. As an alternative, you can specify it in your ``/etc/openhab2/services/addons.cfg`` file as shown below.

{% highlight console %}
# A comma-separated list of bindings to install (e.g. "sonos,knx,zwave")
binding = homematic
{% endhighlight %}

Once the binding is installed, it's time to add the CCU2 gateway as openHAB thing. According to the [binding's readme][oh-homematic-binding], gateway discovery is only supported for Homegear. Therefore we need to add the CCU2 as bridge in a thing file manually. Our example uses the file ``/etc/openhab2/things/test.things``. Please change the gateway address according to your settings.

{% highlight console %}
Bridge homematic:bridge:ccu [ gatewayAddress="192.168.0.55" ]
{% endhighlight %}

After the CCU2 is registered in openHAB, the auto discovery mechanism should find all devices connected to the gateway automatically. They are shown in the inbox of the Paper UI similar to the screenshot below.

![Automatically discovered Homematic device shown in inbox of openHAB]({{ site.url }}/assets/oh_homematic_discovery_result.png){: .center-image }

You can use the Paper UI to add each discovered device as a thing. Just click on the inbox entry, enter a name and click on the add button. If you prefer the file based definition, you can add the device entry in your thing file (here it is located in ``/etc/openhab2/things/test.things``) as shown below.

{% highlight console %}
Bridge homematic:bridge:ccu [ gatewayAddress="192.168.0.55" ]
{
  Thing HM-CC-RT-DN MEQ0799157
}
{% endhighlight %}

# Basic openHAB configuration

The Homematic binding recognizes available channels out of the box. They are shown in the Paper UI if you navigate to things entry in the configuration section. Clicking on the thing entry shows detailed information. This includes the most common channels. 

Additionally, there is a "Show more" button in the upper right corner. Clicking on it shows all available channels of the thermostat.

![Homematic HM-CC-RT-DN thermostat channels recognized by the Homematic openHAB binding]({{ site.url }}/assets/oh_homematic_channels.png){: .center-image }

The channel names give a hint regarding their purpose. For getting a deeper understanding, I recommend having a look at the documents listed on [HomeMatic-INSIDE][oh-homematic-list]. Regarding the channel specification, especially [the fourth document containing the data point specification][oh-homematic-channels] is relevant.

Using those channels is well known openHAB configuration stuff. You can create items assigned to relevant channels for showing or modifying the values via Basic UI using a sitemap. For this reason, no further explanation is given here. A file based item configuration is shown below.

{% highlight console %}
Number  Current_Temperature "Current temperature [%.1f °C]"  <temperature>   { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#ACTUAL_TEMPERATURE" }
Number  Target_Temperature  "Target temperature [%.1f °C]"   <temperature>   { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#SET_TEMPERATURE" }
String  Control_Mode        "Mode [%s]"                      <settings>      { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#CONTROL_MODE" }
Number  Battery_State       "Battery state [%.1f V]"         <energy>        { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#BATTERY_STATE" }
Number  Valve_State          "Valve state [%.0f %%]"         <settings>      { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#VALVE_STATE" }
Number  Boost_Countdown      "Boost countdown [%.0f min]"    <settings>      { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#BOOST_STATE" }
{% endhighlight %}

For showing this in the Basic UI, a corresponding sitemap definition is needed. Here we show the most common values and offer the possibility to change the target temperature. Please have a look at the example sitemap shown below.

{% highlight console %}
sitemap test label="Homematic thermostat test"
{
    Frame label="Homematic thermostat" {
        Text item=Current_Temperature
        Setpoint item=Target_Temperature step=0.5 minValue=17 maxValue=26
        Text item=Control_Mode
        Text item=Battery_State
        Text item=Boost_Countdown
        Text item=Valve_State
    }
}
{% endhighlight %}

Now you can see the current state of the thermostat's battery, the current temperature, and the currently active control mode. Additionally you are able to change the target temperature by clicking the up and down buttons. A screenshot is attached below. 

![Simple openHAB configuration shown in Basic UI]({{ site.url }}/assets/oh_homematic_simple_basicui.png){: .center-image }

If you'd like to show different labels for the control modes, you can use the [Map transformation service][oh-transformation-map]. For further details please have a look at the [documentation][oh-transformation-map].

One important point is the control mode, which is shown as read-only value here. According to the [Homematic HM-CC-RT-DN thermostat manual][thermostat-manual], it means the following:
* If the automatic mode (``AUTO-MODE``) is used, the target temperature is controlled by the thermostat or the CCU2 according to the configured schedule. We could call it the Homematic controlled mode.
* If the manual mode (``MANU-MODE``) is used, the target temperature stays as is. It won't be modified automatically by the thermostat or by the CCU2 any more. We could call it the user or openHAB controlled mode.
* The boost mode (``BOOST-MODE``) causes the thermostat to open the valve to a defined percentage (default is 80%) for a limited period (default is 5 minutes). It's purpose is to heat up cold rooms as fast as possible.
* The party mode (``PARTY-MODE``) holds a fixed temperature for a given time period. It's purpose is to disable the regular schedule, e.g. if you are on holiday. Once the end of the period is reached, the automatic mode becomes active again and the regular schedule is continued.

The difference bettwen the automatic and manual mode might become more clear, if we talk about an example. The user configured a schedule on the CCU2 (set the target temperature to 18°C between 10 p.m. and 5 a.m. and set it to 21°C the rest of the day), which was transferred to the thermostat. The user or an external application like openHAB changed the target temperature to 24°C on 9 p.m. 
* If the automatic mode is used, the target temperature is 18°C on 11 p.m. 
* If the manual mode is active, the target temperature is still 24°C on 11 p.m.

The consequence is, that we should use the manual mode, if we'd like to use openHAB for effectively scheduling the target temperatures. For this reason, you should think about changing the mode of your thermostat to the manual one using the CCU2 or the button of the thermostat.

The party mode is not relevant for us, because this is just a change in the schedule - and this should be handled by openHAB in the future.

# Extended openHAB configuration

The extended configuration allows us to change the control mode via openHAB. For reaching this, we need to dig a bit deeper. As the [data point specification of Homematic][oh-homematic-channels] shows, the control mode (``CONTROL_MODE``) is a read-only data point. This means, we can read the value using the corresponding channel, but we are not able to change it using the same channel.

For being able to switch between the automatic and the manual mode, we need to use the following data points:
* The automatic mode is triggered by sending anything to the data point ``AUTO_MODE``. In our example, this corresponds to the channel ``homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#AUTO_MODE``.
* The manual mode is triggered by sending the target temperature to the data point ``MANU_MODE``. In our example, this corresponds to the channel ``homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#MANU_MODE``.
* The boost mode is triggered by sending anything to the data point ``BOOST_MODE``. In our example, this corresponds to the channel ``homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#BOOST_MODE``.

Due to the fact, that each item is assigned to a single channel in openHAB, additional items need to be defined for being able to trigger a mode switch. In case of a file based item definition, it would look like shown below.

{% highlight console %}
Number  Current_Temperature  "Current temperature [%.1f °C]" <temperature>   { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#ACTUAL_TEMPERATURE" }
Number  Target_Temperature   "Target temperature [%.1f °C]"  <temperature>   { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#SET_TEMPERATURE" }
String  Control_Mode_CCU2    "Mode CCU2 [%s]"                <settings>      { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#CONTROL_MODE" }
Number  Control_Mode_openHAB "Mode openHAB[%s]"             <settings>
Number  Control_Mode_Manu    "Manual mode"                  <settings>       { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#MANU_MODE" }
Switch  ControlMode_Auto     "Automatic mode"                <settings>      { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#AUTO_MODE" }
Switch  Control_Mode_Boost   "Boost mode"                    <settings>      { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#BOOST_MODE" }
Number  Battery_State        "Battery state [%.1f V]"        <energy>        { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#BATTERY_STATE" }
Number  Valve_State          "Valve state [%.0f %%]"         <settings>      { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#VALVE_STATE" }
Number  Boost_Countdown      "Boost countdown [%.0f min]"    <settings>      { channel="homematic:HM-CC-RT-DN:ccu:MEQ0799157:4#BOOST_STATE" }
{% endhighlight %}
 
There is the already defined string item connected to the read-only control mode, which will be used to get the currently set mode of the thermostat (``Control_Mode_CCU2``). A switch item will be used for triggering the automatic mode (``Control_Mode_Auto``) as well as the boost mode (``Control_Mode_Boost``), and a number item (``Control_Mode_Manu``) will be used for sending the target temperature to the data point of the manual mode for activating it. Additionally, there is an unassigned item (``Control_Mode_openHAB``). 

Now we need to connect the items using some logic implemented as rules. The item ``Control_Mode_openHAB`` will be used to store the value selected in openHAB. The rule needs to consider the following things:
1. If ``Control_Mode_openHAB`` changed to the manual mode, the value of ``Target_Temperature`` is sent to ``Control_Mode_Manu``.
1. If ``Control_Mode_openHAB`` changed to the automatic mode, the value ``ON`` (you can use whatever value you want, it's just important to send anything to the datapoint) is sent to ``Control_Mode_Auto``.
1. If ``Control_Mode_openHAB`` changed to the boost mode, the value ``ON`` (you can use whatever value you want, it's just important to send anything to the datapoint) is sent to ``Control_Mode_Boost``.
1. If ``Control_Mode_CCU2`` changed, the value of ``Control_Mode_openHAB`` is changed accordingly. A change is needed, if the control mode is changed outside of openHAB, e.g. via CCU2 web application or directly on the thermostat.

The topics mentioned in #1 to #3 are implemented in the first rule shown below. It reacts on changed values of the openHAB mode selector and triggers the mode specific item and therefore the appropriate Homematic data point.

{% highlight groovy %}
rule "Thermostat mode switcher"
when
    Item Control_Mode_openHAB changed 
then
    logDebug("homematic", "Mode switcher rule of thermostat executed with control mode '{}'", 
        Control_Mode_openHAB.state.toString)
		
if("MANU-MODE".equals(Control_Mode_openHAB.state.toString)) {			
    Control_Mode_Manu.sendCommand(Target_Temperature.state as Number)
    logInfo("homematic", "Switched thermostat to MANU_MODE with value {}", 
        Target_Temperature.state as Number)
} else if("AUTO-MODE".equals(Control_Mode_openHAB.state.toString)) {
    Control_Mode_Auto.sendCommand(ON)
    logInfo("homematic", "Switched thermostat to AUTO_MODE")
} else if("BOOST-MODE".equals(Control_Mode_openHAB.state.toString)) {
    Control_Mode_Boost.sendCommand(ON)
    logInfo("homematic", "Switched thermostat to BOOST_MODE")
}
end
{% endhighlight %}

The second rule covers topic #4. If the mode changes on the CCU2 side, the openHAB item is updated. Surprisingly, there is no infinite loop. It seems like the binding handles incoming commands containing the same item value that is already set. For being safe, you could add persistence and compare the new value with the previous one - but we won't cover this here.

{% highlight groovy %}
rule "Sync thermostat mode"
when
    Item Control_Mode_CCU2 changed
then
    logDebug("homematic", "New control mode sent by CCU2 is '{}'", 
        Control_Mode_CCU2.state.toString);
    Control_Mode_openHAB.sendCommand(Control_Mode_CCU2.state.toString)
end
{% endhighlight %}

Last but not least, we need to update the sitemap definition. The text only showing the currently set control mode (as it was configured for the basic setup), needs to be replaced by a selection item. It offers the possibility to choose between the automatic, the manual and the boost mode.

{% highlight console %}
sitemap test label="Homematic thermostat test"
{
    Frame label="Homematic thermostat" {
        Text item=Current_Temperature
        Setpoint item=Target_Temperature step=0.5 minValue=17 maxValue=26
        Selection item=Control_Mode_openHAB mappings=["AUTO-MODE"="Automatic", "MANU-MODE"="Manual", "BOOST-MODE"="Boost"]
        Text item=Battery_State
        Text item=Boost_Countdown
        Text item=Valve_State
    }
}
{% endhighlight %}

This finally leads to a Basic UI thermostat representation like shown below. It now offers the possibilty to change the thermostat's control mode manually.

![Extended openHAB configuration shown in Basic UI]({{ site.url }}/assets/oh_homematic_extended_basicui.png){: .center-image }

# Connection troubleshooting

While developing the configuration, I had some trouble establishing a connection from my Windows laptop to the Homematic CCU2 gateway. Therefore I'll describe some of the problems and the solutions, which solved the problem in my case.
 
The first problem was, that there are calls from the CCU2 to your computer. For this reason, the Homematic binding needs to open some callback ports. This callback connection was not possible, because my firewall blocked those requests. Whitelisting those ports solved the problem. If you didn't change the binding's default configuration, the relevant ports are ``9125`` and ``9126``.

The second issue was related to the standby mode of my laptop. After it booted from standby mode, the communication didn't work any more. A complete reboot solved the problem. 

Third, it might be the case, that one of those ports is already used. The thing details of the bridge in the Paper UI will show an error message mentioning already used ports. If the log level is set accordingly, you will find a message similar to ``java.net.BindException: Address already in use`` in your log file. In this case, you can choose other ports by modifying your thing configuration. This is described in the [readme][oh-homematic-binding] and shown in the example below.

{% highlight console %}
Bridge homematic:bridge:ccu [ gatewayAddress="192.168.0.55", xmlCallbackPort=9127, binCallbackPort=9126 ]
{% endhighlight %}

If you are facing a problem, which is not covered above, it might help to set a more verbose log level for the binding (see [openHAB documentation][oh-logging]). If you'd like to set the configuration just for the current process, you can use the Karaf console (see also the logging related chapter of [this blog post][post-zwave]).

{% highlight terminal %}
openhab> log:set debug org.openhab.binding.homematic
{% endhighlight %}

If you'd like to change the log level persistently, you can change the logging configuration in ``/var/lib/openhab2/etc/org.ops4j.pax.logging.cfg``. For setting the Homematic binding log level to debug, you should add a line similar to the one shown below or change the existing one if present.

{% highlight terminal %}
log4j.logger.org.openhab.binding.homematic = DEBUG
{% endhighlight %}

From this time on, more Homematic related log messages are written to the log file, which is located in ``/var/log/openhab2/openhab.log``. For more details, please have a look at the [corresponding section of the binding's readme][oh-homematic-logging].

# Conclusion

This blog post described, how the Homematic HM-CC-RT-DN thermostat can be integrated in an openHAB smart home environment by using the Homematic CCU2 gateway. The Homematic binding is easy to use and offers access to all data points offered by the device. The basic configuration for showing the current temperature and setting the target temperature in the Basic UI is no big deal resp. regular openHAB configuration.

But if you'd like to use some extended functionality, you need to deal with multiple data points using some logic implemented as rules. Therefore I guess, a working example is a good starting point for building your own configuration.

The scheduling part was not covered here. In general, you should set the control mode to the manual one. The scheduling itself can be implemented using [rules][oh-rules] or something like the [Google Calendar Scheduler][oh-gcal]. Maybe this will be the topic of another blog post.
	
[oh-homepage]: https://www.openhab.org
[oh-paths]: http://docs.openhab.org/installation/linux.html#file-locations
[oh-homematic-binding]: http://docs.openhab.org/addons/bindings/homematic/readme.html
[post-zwave]: {{ site.baseurl }}{% post_url 2017-10-30-zwave-debugging-in-openhab %}
[oh-logging]: http://docs.openhab.org/administration/logging.html
[oh-homematic-logging]: http://docs.openhab.org/addons/bindings/homematic/readme.html#debugging-and-tracing
[oh-homematic-list]: https://www.homematic-inside.de/software/download/item/homematic-skript
[oh-homematic-channels]: https://www.homematic-inside.de/software/download?task=callelement&format=raw&item_id=132&element=8820adb0-12fa-4946-a761-c642fcd52f7b&method=download&args[0]=3.0&args[1]=ca135f3b2e295cee1af6cd4d74d73b83
[oh-transformation-map]: http://docs.openhab.org/addons/transformations/map/readme.html
[thermostat-manual]: http://www.eq-3.de/Downloads/eq3/downloads_produktkatalog/homematic/bda/HM-CC-RT-DN_UM_GE_eQ-3_web.pdf
[oh-rules]: http://docs.openhab.org/tutorials/beginner/rules.html
[oh-gcal]: http://docs.openhab.org/addons/ios/gcal/readme.html