---
layout: post
title:  "Using The openHAB Thing Status"
category: IT
tags: openhab smart-home
bigimg: /img/bigimg_happen.jpg
credname: Bich Tran
credurl: https://www.pexels.com/de-de/foto/daten-dokument-graph-hinweis-669986/
comments: true
excerpt: For several use cases it would be beneficial to have access to the openHAB thing status. Unfortunately the thing status can not easily be assigned to an item. This blog post explains a workaround.
---

Recently I recognized that some of the temperature and humidity sensors of my openHAB setup get unavailable from time to time.
I had the idea to show the availability status of the connected openHAB things in the sitemap and to send this data also to my monitoring setup based on InfluxDB and Grafana.

Initially I thought that it should not be too difficult to provide this thing status in an item, because at least the Paper UI already shows the status of connected things.
Unfortunately I was wrong.

# No Common Channels Provided By All Things

Basically data is provided by things.
The datapoints itself are called channels which can be connected to items.
Such items can then be used in sitemaps or for sending data to a datastore using the persistence functionality provided by openHAB.

The main problem for the things status use case is that there are no general openHAB channels available for all things.
Channels are provided by the binding.
Therefore it is possible that one or the other binding provides things with a thing status channel - but this would only be the case for such specific bindings.

That's why I would like to explain a binding independent workaround, which makes it possible to provide the current thing status in an item.

# Using Rules As Workaround

The missing link between the thing and the item can be established with a rule.
General documentation about how rules work is provided in the [openHAB documentation][oh-rules].

As a preparation it is necessary to create a new item which will contain the thing status later.
This is the item we can then use for sitemaps or for persistence purposes.

```
Switch   Bathroom_Sensor_Status   "Status"   <switch>
```

To set the thing status as value of this item, a new rule is created.
This rule defines a [thing based trigger][oh-thing-trigger] and is therefore executed whenever the things status changes.
In the script block of the rule we take care of setting the item value according to the thing status.

```groovy
rule "Bathroom Thing Status"
when
  Thing "jeelink:lacrosse:30" changed 
then
    var thingStatusInfo = getThingStatusInfo("jeelink:lacrosse:30")

    if ((thingStatusInfo !== null) && (thingStatusInfo.getStatus().toString() == "ONLINE")) {
        Bathroom_Sensor_Status.postUpdate(ON)
    } else {
        Bathroom_Sensor_Status.postUpdate(OFF)
    }
end
```

From now on every change of the thing status is reflected in the item.
Please keep in mind that the first sync of the thing status will happen with the first change.
You therefore have to restart openHAB once you defined the item and the rule.

If you don't want to restart openHAB, you could add a second trigger - e.g. a [time based one][oh-time-trigger].

# Conclusion

Unfortunately there is no easy, built-in solution to provide the thing status in an item - but it is still possible to reach this goal with a workaround.

You can provide a rule which reacts on thing status changes and takes care of setting the item value according to the new thing status.

[oh-rules]: https://www.openhab.org/docs/configuration/rules-dsl.html
[oh-thing-trigger]: https://www.openhab.org/docs/configuration/rules-dsl.html#thing-based-triggers
[oh-time-trigger]: https://www.openhab.org/docs/configuration/rules-dsl.html#time-based-triggers