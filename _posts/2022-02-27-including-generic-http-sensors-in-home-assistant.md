---
layout: post
title:  "Including Generic HTTP Sensors in Home Assistant"
category: IT
tags:
    - smart-home
    - home-assistant
bigimg: /img/bigimg_connect.jpeg
credname: Brett Sayles
credurl: https://www.pexels.com/de-de/foto/bau-industrie-kabel-linie-4508748/
comments: true
excerpt: While switching from openHAB to Home Assistant, I had to migrate a custom binding to read some data from a heater. Using the Home Assistant RESTful integration allowed including this thing using pure configuration and made the migration easy - without an integration being available and without custom implementation.
---

For many years, [openHAB][openhab-website] was my preferred smart home solution.
Before upgrading to openHAB 3, I had another look on alternatives and gave [Home Assistant][home-assistant-website] a try.
I was really impressed, how easy it is to use and decided to migrate from openHAB to Home Assistant.

The migration included a custom openHAB binding I used to gather some data from a heater, which exposes data via an HTTP interface.
Similar to openHAB, there is no Home Assistant integration for this kind of thing available.
I tried to avoid implementing a custom integration for now and ended up using the [RESTful integration][rest-integration], which allows including generic sensors with pure configuration.

## Context of the Use Case

The HTTP API of the heater is available in the local network and is secured with an API key.
To get the data, an HTTP GET request must be used containing the API key as query parameter.

```bash
curl --location --request GET 'http://<ip-address>/data?key=<api-key>'
```

The response unfortunately contains the data as plaintext, with one sensor value per line.
A shortened example is provided below.

There is another endpoint, which defines the meaning and measurement unit of each line.
From the other request, we e.g. know that the fourth line contains the current boiler temperature in °C (83.61 °C) and that the fifth line contains the current flue gas utilization in % (0.00 %). 

```
 
 
 
83.61
0.00
 
 
0
0
```

We would like those two values to show up in the Home Assistant user interface.

## Setup in Home Assistant

Home Assistant comes with the [RESTful integration][rest-integration], which polls an HTTP API, extracts data and pushes it to a state.
This is exactly what is needed here, if you would not like to implement a custom integration.

Configuration is done in the `configuration.yaml` file, which I edit using the Visual Studio Code Server running as part of Home Assistant (for details about this please have a look at the [community addon][vscode-addon]).

![Editing the configuration.yaml file using the integrated Visual Studio Code Server](/img/ha_rest-vscode.png)

The basic setup should be self-explaining:
* Configure the URL of the API using the `resource` parameter.
* Specify the query parameters using the `params` attribute.
* Define the sensors as a list contained in the `sensor` attribute.

The most important part in my case was parsing the response returned by the API of the heater.
It was easy to find out, that you can define the parsing expression in the `value_template` attribute of each sensor.

The [documentation][templating] explains the templating mechanism in more detail.
My main takeaways are, that it uses [Jinja][jinja-website] as template engine and that Home Assistant provides an integrated tool to develop and test such expressions.

There are some more settings like the `scan_interval` available, which are explained in the documentation.
For the settings related to a single sensor, you find it [here][rest-integration].
All other settings are documented [here][rest-integration-sensor].

Once the configuration is done, you need to validate the configuration and restart Home Assistant, if no errors are reported during validation.
Afterwards, you can use the sensor state to display it in the user interface.

## Conclusion

This blog post explained, how generic sensors providing data via an HTTP API can be added to Home Assistant by using pure configuration.
From my point of view, this is a pretty easy and fast way of adding things, for which there is not yet an integration available.

Nevertheless, having a real integration would feel better.
Therefore, I already added realizing my first custom Home Assistant integration to my todo list.

[rest-integration]: https://www.home-assistant.io/integrations/rest
[rest-integration-sensor]: https://www.home-assistant.io/integrations/sensor.rest
[openhab-website]: https://www.openhab.org/ome
[home-assistant-website]: https://www.home-assistant.io/
[vscode-addon]: https://github.com/hassio-addons/addon-vscode
[templating]: https://www.home-assistant.io/docs/configuration/templating/#processing-incoming-data
[jinja-website]: https://palletsprojects.com/p/jinja/