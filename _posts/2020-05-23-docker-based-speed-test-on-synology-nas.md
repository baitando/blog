---
layout: post
title:  "Docker Based Speed Test On Synology NAS"
category: IT
tags: nas
bigimg: /img/bigimg_speed.jpg
credname: Karol D
credurl: https://www.pexels.com/de-de/foto/abend-autobahn-autos-bewegung-409701/
comments: true
excerpt: Recently I upgraded my internet tariff. To observe the bandwith effect of the upgrade I was searching for a convenient speed test and found an effective solution running a Docker container on my Synology NAS.
---

When I upgraded my internet tariff, I was excited about how much the bandwith will effectively increase.

As a first attempt for getting some insights I used one of the various online speed tests.
This worked pretty well - but the downside is that you actively need to trigger the speed test to get the data for the specific point in time.

To get a better picture of the bandwith situation, my goal was to establish a scheduled speed test which runs once per hour.
The result is a speed test solution running as a Docker container on my Synology DS218+ NAS.
In this blog post I would like to explain how the setup works.

# Prerequisites

The following explanations assume that you are familiar with Docker in general and that you know how to run Docker containers on a Synology NAS.
If the latter is not the case, you could have a look at my blog post [Using Docker on Synology NAS][post-docker-synology].

The general use of the Docker image to run a speed test does of course not depend on a Synology NAS.
That's why it can be used on any platform.
Nevertheless, this blog post will cover the topic with focus on the Synology NAS.

# Download The Docker Image

Searching for speed test Docker images I stumbled accross the [roest/docker-speedtest-analyser][docker-speed-test] image which is publicly available on [Docker Hub][docker-speed-test].
I tried it and came to the conclusion that it perfectly matches my requirements.

The features included are:

* Scheduling is already configured to run hourly, but the schedule can be overridden.
* The results are provided as interactive diagram delivered by the built-in webserver.
* It is possible to persist the results in a file based format.

Under the hood, the Docker image uses the [speedtest-cli] tool written in Python which connects to [speedtest.net](https://www.speedtest.net/).
Thanks to [Tobias Rös][tobias-roes] for providing this convenient speed test Docker image.

To download the Docker image please open the Docker application in the Synology user interface.
Make sure that Docker Hub is the active Docker registry.

![Checking that Docker Hub is the active Docker Registry](/img/docker_speedtest_registry.png)

Search for the Docker image, e.g. by entering `docker-speedtest-analyser` as search term, select the proper Docker image and download it by clicking the button.

A dialog will ask for the tag to download.
I recommend choosing a specific version instead of `latest`.
At the time I wrote the blog post, the most recent version was `1.2`.

![Downloading the roest/docker-speedtest-analyser Docker image](/img/docker_speedtest_image.png)

# Create The Docker Container

To create a new Docker container from the previously downloaded Docker image, please switch to the image section.
Select the Docker image and click the `Launch` button.

A dialog will open in which you can modify the name of the Docker container.
You could proceed with the `Next` button, but I recommend doing some additional settings by clicking on the `Advanced Settings` button.

![Creating the speed test Docker container](/img/docker_speedtest_create.png)

The first useful setting is to enable the auto-restart of the Docker container.
If this is enabled, the Docker container will automatically start with the NAS.

![Enabling the auto-restart functionality of the Docker container](/img/docker_speedtest_autorestart.png)

To keep the speed test results persistent and accessible, you should configure a volume and map it to `/var/www/html/data`.
As described in [another blog post][post-docker-synology], I use a network share for all Docker related things like configuration backups and volumes.
This way you can easily access the volume data.

![Configuring the volumes of the Docker container](/img/docker_speedtest_volume.png)

Additionally, I configured a specific external port for the webserver running in the Docker container.

If you would not set a specific port, a free port is assigned.
This would be a bit cumbersome, if you want to access the speed test result website. 

![Configuring the port of the contained webserver](/img/docker_speedtest_port.png)

Once all settings are done, we can go back and proceed.
An overview of the settings will be shown.
Once the `Apply` button is clicked, the container will start.

![Configuring the port of the contained webserver](/img/docker_speedtest_overview.png)

# Accessing The Results

To access the results, we can use the browser and navigate to `http://<nas-ip-or-domain>:<speedtest-port>`.
Please make sure to replace the values accordingly.

* The `<nas-ip-or-domain>` is the IP address or the domain you use to access your NAS.
* The `<speedtest-port>` is the port you configured for the Docker container in one of the previous steps.

Please note that it could take some time until first results show up.
By default, the speed test runs every full hour.

![Results of the speed test](/img/docker_speedtest_results.png)

If you would like to access the plain data, you can have a look at the folder you mapped as volume.
It will contain a CSV file with the raw speed test result data.

# Optional Steps

There are several optional steps, which can help to further improve the setup.
In my environment I did the following additions steps:

* Create a backup of the Docker configuration of the specfic Docker container.
* Create a separate domain or subdomain to access the speed test results.
* Create a reverse proxy entry to access the speed test results.

These steps are separate topics, which are not further explained here.

# Conclusion

This blog post explained how a scheduled speed test can be set up using Docker on a Synology NAS.
There are similar Docker images available, which solve the same purpose.
So feel free to choose any Docker image which fits to your use case.

[post-docker-synology]: {{ site.baseurl }}{% post_url 2019-09-22-using-docker-on-synology-nas %}
[docker-speed-test]: https://hub.docker.com/r/roest/docker-speedtest-analyser/
[tobias-roes]: https://github.com/roest01
[speedtest-cli]: https://pypi.org/project/speedtest-cli/