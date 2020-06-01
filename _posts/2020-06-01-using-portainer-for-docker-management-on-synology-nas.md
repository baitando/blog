---
layout: post
title:  "Using Portainer For Docker Management On Synology NAS"
category: IT
tags: nas docker
bigimg: /img/bigimg_container_management.jpg
credname: chuttersnap
credurl: https://unsplash.com/photos/kyCNGGKCvyw
comments: true
excerpt: The Docker management user interface of a Synology NAS is great for basic use cases - but it does not cover all features provided by Docker. Portainer can be used as an alternative to unleash the full potential. This blog post explains how to do the setup.
---

Most of my hobby applications run as Docker containers on a Synology NAS and I really love this cheap and easy possibility to run applications at home.

For managing the Docker containers I used the Synology Docker management user interface until now.
It is a lightweight possibility to download Docker images, and to manage the Docker containers without having to deal with the command line.

Sometimes there is a need for settings, which can't be done with the Synology user interface.
This is e.g. the case, if you want to set a specific Docker log driver to collect the logs of Docker containers in a central place - at least this was the trigger for me to check, which alternatives are available.

I already knew [Portainer][portainer-website] but never used it on a NAS which usually comes with some wrappers around well-known tools like Docker, which sometimes lead to problems.
So I decided to give it a try and to document the necessary setup steps in this blog post.

# Create The Portainer Docker Container

Portainer itself will also run as a Docker container which also simplifies the setup.

First I thought that the easiest approach would be creating the Portainer Docker container using the Synology NAS user interface.
I tried it and soon I recognized, that it is not possible to mount something other than network shares as volumes.
Therefore, it is not possible to mount `/var/run/docker.sock` as required.
This means, it is unfortunately not possible to manage the Portainer Docker container using the Synology user interface.

That's why the Portainer Docker container needs to be created using the command line.
Please connect to the NAS via SSH.
Then replace the placeholders in the command below and run the resulting command via SSH on the NAS.

```shell script
sudo docker run -d
  -p <port>:9000
  --name=<name>
  --restart=always
  -v /var/run/docker.sock:/var/run/docker.sock
  -v <volume>:/data
  portainer/portainer:<version>
```

The placeholders used are:
* `port`: The port on which the Portainer web application will be reachable.
Make sure this port is not in use yet.
I used `9000` as value.
* `name`: The name you want to use for the Portainer Docker container.
I used `portainer` as value.
* `volume`: The path to the directory on the NAS which is used to store persistent data belonging to Portainer.
I used `/volume1/docker/portainer/data`.
* `version`: The version of the Portainer Docker image.
I recommend to always specify a specific version to have full control over the version used for the container and to make the setup reproducible.
If this is not relevant for you, feel free to use `latest`.
I used `1.24.0`.

This command creates the Portainer Docker container and takes care, that it is automatically started e.g. if the NAS was rebooted.

# Configure Portainer

Once the Docker container is created and running, you can access the Portainer user interface in the browser.
Just replace the placeholders in `<http|https>://<nas-domain-or-ip>:<port>` and navigate to the resulting URL.

The placeholders used are:
* `<http|https>`: Choose the protocol you use to access the NAS.
* `<nas-domain-or-ip>`: The domain or IP you use to access the NAS.
* `<port>`: The port you used as placeholder in the previous section.

The page you will see should look similar to the one below.
It is used to create the first user on Portainer which will get administrator permissions automatically.
Specify the desired username and a safe password and click on the button to create this user.

![Create the portainer user](/img/portainer_synology_initial_login.png)

The next step is to further configure Portainer.
You need to select the `Local` option.
Then you can click the button to start the connection process.

![Connect to the local Docker daemon](/img/portainer_synology_local.png)

Once connected, you can navigate to the containers section which lists all existing Docker containers.
This includes also Docker containers which where created using the Synology Docker user interface.

Please be aware that the Docker containers won't show up anymore in the Synology user interface once you modified an existing Docker container with Portainer.
Also, all Docker containers created by Portainer won't show up in the Synology user interface. 

![List of existing Docker containers](/img/portainer_synology_containers.png)

# Conclusion

With the Portainer setup described here, you have an alternative to manage Docker containers on a Synology NAS.
Using Portainer offers access to all Docker features - not just the limited feature set provided by the Synology user interface.

[portainer-website]: https://www.portainer.io/