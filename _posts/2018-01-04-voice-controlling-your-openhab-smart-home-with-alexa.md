---
layout: post
title:  "Voice controlling your openHAB smart home with Alexa"
category: IT
tags: smart-home openhab alexa
comments: true
excerpt: Voice controlled home assistants become more and more popular. Let's have a look, how Amazon Alexa can be integrated with openHAB for voice controlling your smart home.
---

It's obvious, that voice controlled home assistants like Google Home and Amazon Alexa gain more and more popularity. I was interested in integrating the latter with my smart home environment based on [openHAB][oh-homepage]. An appropriate [Alexa skill][amazon-alexa-skill] incl. [documentation][oh-alexa-skill] is already available . Therefore I thought, that establishing the connection between Alexa and openHAB could be an easy task. I was wrong. For this reason, I'd like to share my experience for maybe helping others dealing with the same task. 

This blog post assumes, that the openHAB environment is already set up and connected to [myopenhab.org][my-oh]. If the latter is not yet done, you should have a look at one of my [previous blog posts][post-my-oh]. It is also required, that your Alexa device (e.g. the Echo Dot) is installed and working.

All file system paths are valid for an APT based installation. This is e.g. the case, if your instance is hosted on Raspbian and you used the available installation package. For further details please have a look at the [documentation][oh-paths]. The blog post generally refers to openHAB version 2.2.

# Prepare your openHAB instance

First of all we will prepare the openHAB environment. The openHAB Alexa skill requires some meta data for being able to seamlessly integrate into the Alexa smart home section. According to the [documentation][oh-alexa-skill], the item configuration syntax of the [Homekit binding][homekit-docs] is used. We will have a look at some examples.

Due to the fact, that you can have multiple items assigned to the same channel, it is recommended not to modify your existing item definition. Better create new, Alexa specific items for being able to serve the requirements of Alexa and the openHAB Alexa skill. The examples below follow this recommendation and show the original definition and the new one after the Alexa specific additions.

## Thermometers

{% highlight console %}
Number LivingRoom_Temp "Living room [%.0f °C]" <temperature> {channel="..."}
{% endhighlight %}

The item definition above is the original one of a thermometer located in my living room. If we add ``["CurrentTemperature"]`` to the item definition, the Alexa skill will tell Alexa, that this item provides the current temperature. The final definition is shown below. 

{% highlight console %}
Number LivingRoom_Temp "Living room [%.0f °C]" <temperature> {channel="..."}
Number Alexa_LivingRoom_Temp "Living room" <temperature> ["CurrentTemperature"] {channel="..."}
{% endhighlight %}

## Thermostats

{% highlight console %}
Number BathRoom_Current_Temp "Bath room current [%.0f °C]" <temperature> {channel="..."}
Number BathRoom_Target_Temp "Bath room target [%.0f °C]" <temperature> {channel="..."}
{% endhighlight %}

The item definition above is the original one of a thermostat located in the bath room. In contrast to the thermometer definition, there is an additional target temperature. First we need to change the item defintion of the current temperature similar to the thermometer case. Afterwards, we add ``["TargetTemperature"]`` to the target temperature item.

For enabling Alexa to recognize those two items as a single thermostat, we need to introduce an additional group including ``["Thermostat"]``. Both temperature items need to be assigned to this new group. The final definition is shown below.

{% highlight console %}
Number BathRoom_Current_Temp "Bath room current [%.0f °C]" <temperature> {channel="..."}
Number BathRoom_Target_Temp "Bath room target [%.0f °C]" <temperature> {channel="..."}

Group gAlexa_BathRoom_Thermostat "Bath room thermostat" [ "Thermostat" ]
Number Alexa_BathRoom_Current_Temp "Bath room current [%.0f °C]" <temperature> (gAlexa_BathRoom_Thermostat) [ "CurrentTemperature" ] {channel="..."}
Number Alexa_BathRoom_Target_Temp "Bath room target [%.0f °C]" <temperature> (gAlexa_BathRoom_Thermostat) [ "TargetTemperature" ] {channel="..."}
{% endhighlight %}

## Wall plugs and other switches

{% highlight console %}
Switch Alexa_CoffeeMachine_WallPlug "Coffee machine" { channel="..." }
{% endhighlight %}

The item definition above is the original one of the cofee machine's wall plug. If we add ``["Switchable"]`` to the item definition, the Alexa skill will tell Alexa, that this item can be enabled and disabled. The final definition is shown below.

{% highlight console %}
Switch Alexa_CoffeeMachine_WallPlug "Coffee machine" [ "Switchable"] { channel="..." }
{% endhighlight %}

This works for all openHAB ``Switch`` items. I'm using it e.g. for wall plugs and the power state and mute mode of the TV.

# Enable and configure the openHAB skill

Once your openHAB instance is configured according to the skill's requirements, we need to enable the openHAB Alexa skill. You can enable it in the same way than any other Alexa skill.

![Search for the openHAB skill in the Alexa web application]({{ site.url }}/assets/alexa_oh_skill_search.png){: .center-image }

The screenshots provided are taken from my browser. As an alternative, you can use the iOS or Android app. The process of enabling the skill is similar.

Please open the skills section and search for "openHAB". The search result should contain the relevant entry. Just click on it to show the skill's details.

![Enable the openHAB skill in the Alexa web application]({{ site.url }}/assets/alexa_oh_skill_enable.png){: .center-image }

The detail page offers the possibility to enable the skill. Please click on this button. This opens a pop up browser window, which is showing [myopenhab.org][my-oh]. After logging in, you need to grant Alexa the permission to access your openHAB environment via openHAB Cloud resp. [myopenhab.org][my-oh]. 

![Grant Alexa the permission to access your myopenhab.org account]({{ site.url }}/assets/alexa_oh_skill_oauth.png){: .center-image }

After you clicked on the accept button, a screen similar to the one below is shown. You can close this browser window now. 

![Configuration of myopenhab.org for Alexa finished]({{ site.url }}/assets/alexa_oh_skill_connected.png){: .center-image }

Once you are back on the Alexa page, a dialog offering device discovery should be shown. Please click on the discover button.

![Discover smart home devices in the Alexa web application]({{ site.url }}/assets/alexa_oh_skill_discover_devices.png){: .center-image }

After the discovery process finished, you should see a list of discovered devices. It should contain the items you configured in openHAB earlier.

![List of discovered smart home devices in the Alexa web application]({{ site.url }}/assets/alexa_oh_discovered_devices.png){: .center-image }

# Voice controlling discovered devices

Now you are ready to give voice commands to your openHAB controlled devices via Alexa. The table below gives some examples in English and German. The German commands were tested in my smart home environment. The English equivalent was derived from the German one and fits to the commands mentioned in the [openHAB Alexa skill announcement][announcement] and the [documentation][oh-alexa-skill].

Please keep in mind, that you need to use the names given in your item configuration when speaking to Alexa. English examples use the names defined in the item configuration above. The German examples assume, that you rename the items accordingly ("Living room" in "Wohnzimmer", "Bath room" in "Badezimmer" and "Coffee machine" in Kaffeemaschine)

| Device type | German examples | English examples |
| -------|--------------|-----|
| Thermometers | Alexa, wie ist die Temperatur im Wohnzimmer? | Alexa, what’s the temperature in the living room?|
| Thermostats | Alexa, wie ist die Temperatur im Badezimmer? | Alexa, what's the temperature in the bath room? |
| Thermostats | Alexa, setze die Temperatur im Badezimmer auf 24 Grad. | Alexa, set the bath room temperature to 24 degrees. |
| Thermostats | Alexa, erhöhe die Temperatur im Badezimmer um 2 Grad. | Alexa, increase the bath room temperature by 2 degrees. |
| Thermostats | Alexa, reduziere die Temperatur im Badezimmer um 2 Grad. | Alexa, decrease the bath room temperature by 2 degrees. |
| Wall plugs and other switches | Alexa, schalte die Kaffeemaschine an.| Alexa, turn on the coffee machine. |
| Wall plugs and other switches | Alexa, schalte die Kaffeemaschine aus.| Alexa, turn off the coffee machine. |

# Limitations

The thermometers I'm using additionally provide data regarding the humidity. I found out, that this is currently not supported, as [this thread][humidity-thread] describes.

# Conclusion

Using voice controlled assistants like Amazon Alexa for interacting with a smart home is a great step forward. This blog post has shown the basic setup steps for connecting an openHAB smart home with Alexa. This might seem to be a bit tricky. Therefore I recommend to stick to the examples in this blog post or in the [documentation of this Alexa skill][oh-alexa-skill].

[oh-paths]: http://docs.openhab.org/installation/linux.html#file-locations
[oh-homepage]: http://www.openhab.org/
[my-oh]: https://myopenhab.org/
[post-my-oh]: {{ site.baseurl }}{% post_url 2017-11-04-enable-remote-access-to-a-local-openhab-instance %}
[announcement]: https://community.openhab.org/t/official-alexa-smart-home-skill-for-openhab-2/23533
[homekit-docs]: http://docs.openhab.org/addons/ios/homekit/readme.html
[oh-alexa-skill]: http://docs.openhab.org/addons/ios/alexa-skill/readme.html
[amazon-alexa-skill]: https://www.amazon.de/openHAB-Foundation/dp/B01MTY7Z5L
[homekit]: http://docs.openhab.org/addons/ios/homekit/readme.html
[humidity-thread]: https://community.openhab.org/t/alexa-control-heating/32066/2