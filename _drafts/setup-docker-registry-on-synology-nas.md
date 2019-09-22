---
layout: post
title:  "Running a Private Docker Registry on Synology NAS"
category: IT
tags: nas docker
bigimg: /img/bigimg_notification.jpeg
credname: Snapwire
credurl: https://www.pexels.com/photo/beach-bottle-cold-daylight-292426/
comments: true
excerpt: Synology NAS offers Docker support which can be used to run own containers. This post describes, how you can make use of this to run your own private Docker registry on your NAS.
---

For some years I operated my own server in my private flat as network file storage and to host some of my hobby developer projects as well as the necessary software development tooling.
When I replaced this server with a Synology NAS I decided to run my own Docker registry on this NAS.

In this post I would like to share my experiences related to the setup. 
It is based on a Synology DS218+ NAS which runs the DSM operating system in version 6.2.2-24922 Update 3. 

# Motivation for Docker on the NAS

As time passed on I lost the desire to take care of the maintenance for the server I operated at home.
To simplify things for me, I replaced it with a simple NAS, which is pretty sufficient for backups and file sharing within my network.
It was no big deal to find a solution for the software development tooling, because there are plenty of Software as a Service offerings which are not too expensive or even for free. These measures already reduced the ongoing maintenance effort to nearly zero - which is obviously great.

The last remaining question was what I would do with some databases and how I could run some web applications I implemented myself.
This software is only used in my private network, i.e. there is no compelling reason to run that outside.
I remembered my colleague, who runs a containerized openHAB instance as his smart home solution on his NAS (for further details on this I recommend you to have a look at his [blog post][oh-docker-fabian]).

As a software engineer I already used Docker but I was skeptical if a NAS is sufficient to run Docker itself and multiple containers. 
Due to the fact that my NAS is permanently up and running I decided that it's definitely worth a try.
If this would work, this would safe me money and effort.

# Prepare the NAS

First of all I installed the Docker using the Synology marketplace.
If you search for Docker you will soon find the package.
Then just click on install.

We will need a place to store the persistent data of the Docker containers.
For this purpose I recommend to create a shared folder using the Synology user interface.
Creating such a shared folder should be pretty straightforward if you own and use a NAS.

# Create Containers from Public Images

To create a MySQL Docker container open the Docker app, go to the registry section and and search for the image.
Then double-click on the entry in the list and select the desired version of the image you would like to use.

![Docker Detail Page in Synology Marketplace after Installation](/img/docker_registry_mysql.png){: .center-image }

Then go to the image section, select the image you want to use for the container and click on launch.
Enter the name you want to use for the new container.
If you want to further configure the container (e.g. port mappings, volumes, networks etc.), you can do that by clicking on the button for advanced settings.
We will have to do several of these advanced settings.

The first setting is related to the restart behavior.
I want the container to be automatically started.
Therefore the checkbox needs to be enabled.

![Enable Automatic Restarts for the Container](/img/docker_registry_db_restart.png){: .center-image }

The next setting is the definition of the volumes to use.
In case of a MySQL database we will have to mount the directory `` inside the container to a directory on the NAS.
This can be achieved in the volumes section of the dialog.
There you can create and pick an appropriate folder.
As already mentioned I recommend to use a separate shared folder to store all Docker related things.
In this shared folder I usually create a folder per container.

![Docker Detail Page in Synology Marketplace after Installation](/img/docker_registry_db_volume.png){: .center-image }

I'm pretty sure that you will need this, e.g. to set the MySQL credentials as environment variables of your container.
I won't go into details about this, because this is not specific for the Synology NAS but general Docker configuration.

![Docker Detail Page in Synology Marketplace after Installation](/img/docker_registry_mysql_create.png){: .center-image }

The purpose of my first MySQL container is to act as database for my openHAB instance.
I needed some more databases for my setup and also some other containers created from publicly available images.
This should be easy to reproduce.
Therefore it's not necessary to repeat this here.

The nice thing is that there is an overview section available in the Synology user interface.
It shows all containers and some details together with information about the current resource utilization in total and per container.

![MySQL Database Docker Container](/img/docker_registry_db_dashboard.png){: .center-image }

# Handling of Private Images

The first part related to public images was pretty easy and straightforward, but it does not cover my self-implemented web applications.
Currently they are deployed directly on the server, but it was no problem to create appropriate Docker images to switch to a Docker based deployment and operation model.
This requires a Docker registry to which the images are pushed after the web application was built.

There are several options for handling the Docker registry.
This includes the following ones, which I investigated:

1. Publish a public image on [Docker Hub][docker-hub].
This is possible for free with no relevant restriction related to the image count.

2. Publish a private image on [Docker Hub][docker-hub].
According to the [pricing page][docker-hub-pricing] the free plan is limited to one private repository.
Here it is important to know, that a repository can consist of many versions and flavors of the same image.
To make it easier to memorize this means, you can push the images of a single private application to Docker Hub for free. 

3. Run a private Docker registry like described in the [docs][docker-registry-docs].
There is an [official image][docker-registry-image] available which offers the possibility to run the custom Docker registry itself as Docker container.

Due to the fact that my hobby projects are not intended to be published, it is not a valid alternative for me to provide my Docker images publicly on Docker Hub.
The second option in general sounds good, but I have more than one application.
The free plan would therefore not be sufficient for my needs.
I decided that the smallest paid plan will be my fallback solution.

My preferred solution is therefore to host my own Docker registry as Docker container on my NAS.

# Create a Registry Container

That said the first step is to create a new container from the Docker registry image.
To create the registry you can easily adapt the process described for MySQL.

Search for the registry image in the registry, pick the proper version and create a container.
I recommend to go to advanced settings and make sure, that the automatic restart is enabled.
Another setting I did is the local port of the container port 5000.
I mapped it to 9160, but of course this is up to which port you want to use.

Now we have a running Docker registry.
To check whether it works, you can call the URL `http://<nas_ip_or_domain>:<configured_port>/v2/_catalog`.
You should get a JSON result with an empty `repositories` array.

# Registry without Transport Level Security

This could already be sufficient, but this registry has some downsides.
Docker in general assumes that the registry serves its content via https, but our registry is only available via http right now.
If we execute the command `docker pull <nas_ip_or_host>:<configured_port>/doesnotexistyet:latest` you will get a response similar to `Error response from daemon: Get https://<nas_ip_or_host>:<configured_port>/v2/: http: server gave HTTP response to HTTPS client`.

It is possible to work with such http served registries (Docker calls that insecure registries), but you would have to explicitly configure the Docker daemon to trust this specific insecure registry.
The Docker documentation about [insecure registries][docker-insecure] explains some details and also points out the consequences of using an insecure registry.

If would like to follow my recommendation of securing the registry you can directly jump to the next headline.
If you want to stick to an insecure registry, you will have to do some configuration on the commandline.
This will now be described in more detail.

The Synology NAS allows to configure a Docker registry with a http URL and is also able to list the images located in this registry.
But as soon as you try to pull an image by double-clicking on it, this will fail.

To make this work we need to modify the `/var/packages/Docker/etc/dockerd.json` which is the equivalent of the `daemon.json` file mentioned in the Docker documentation.
The configuration is exactly what is described in the [insecure registries documentation][docker-insecure].
This means you have to update the file and add the `insecure-registries` part.

```json
{
   "data-root" : "/var/packages/Docker/target/docker",
   "log-driver" : "db",
   "registry-mirrors" : [],

   "insecure-registries" : [
        "<nas_ip_or_domain>:<configured_port>"
      ]
}
```

After you updated this file you have to restart the Docker daemon.
On the Synology NAS you can do that either via the user interface in the package manager section or via commandline by using the `synoservice --restart pkgctl-Docker` command.

The necessary configuration on other clients is pretty much the same.
This is e.g. necessary for the computer on which the Docker image will be built and pushed to  the registry.
In case of Mac or Linux the file is called `daemon.json` and located in the `~/.docker` directory.

If you use Docker Desktop for Mac or Windows, you can also use the provided user interface.
Just open preferences, go to the daemon section and add your insecure registry to the list.
After you did the change please also restart the Docker daemon.

If you then execute the command `docker pull <nas_ip_or_host>:<configured_port>/doesnotexistyet:latest` again, you should see an error message similar to `Error response from daemon: manifest for <nas_ip_or_host>:<configured_port>/doesnotexistyet:latest not found: manifest unknown: manifest unknown`.

This is a good message, because there is indeed no such image available in our brand new Docker registry.
The configuration of the insecure registry worked and the registry is available.

# Registry with Transport Level Security

I definitely recommend to use transport level security for the Docker registry.
In this case the registry is not listed as insecure registry, i.e. if you already added the registry as an insecure one you will have to revert this.

What I explain here requires that you already set up a domain for your NAS or a specific domain only for your registry, and that you configured a proper certificate for the domain to use.
You can either use a bought certificate, a free one e.g. issued by Let's Encrypt or a self-signed certificate.
Synology provides some information on this topic in the [documentation][synology-certs].

To make the registry secure, we put a reverse proxy in front of it, which will take care of transport level security incl. certificate handling.
We can use the application portal which is provided with the Synology NAS.
You find it in the control panel in the menu entry application portal.
Then got to the reverse proxy section and create a new entry.

![Reverse Proxy Configuration for Private Docker Registry](/img/docker_registry_reverse_proxy.png){: .center-image }

For the source configuration make sure to select `https` as protocol, enter the ip or hostname of your NAS and select `443` as port.
In the destination section we use `localhost` and the port we configured for our registry Docker container.

Please open a terminal session on your NAS.
If you execute the command below on your NAS and you see an error message similar to `Error response from daemon: manifest for <nas_host>:<configured_port>/doesnotexistyet:latest not found: manifest unknown: manifest unknown` everything is up and running fine.

```bash
docker pull <nas_ip_or_host>:<configured_port>/doesnotexistyet:latest
```

If you see an error message similar to `Error response from daemon: Get https://<nas_host>:<configured_port>/v2/: x509: certificate signed by unknown authority` we have to do some more steps.
This error is typical if you use a private certification authority which was e.g. created by yourself.
We can solve this by adding the certificate of the certification authority to the folder `/var/packages/Docker/etc/certs.d/<nas_host>:<configured_port>`.
The Docker daemon uses the files in this directory to validate the certificate of the registry.

Please provide the certificate of your certification authority in the folder `/var/packages/Docker/etc/certs.d/<nas_host>:<configured_port>`.
Docker will pick it up when it verifies the certificate of your registry.

If you don't have the certificate available you can simply download the complete chain with a nice command I found in a [StackExchange post][openssl-stackexchange].
The command stores each certificate in a separate file.
Watch out for the file with the highest number, because it contains the certificate of your certification authority.
It is already PEM encoded which is perfect for our purpose.

```
openssl s_client -showcerts -verify 5 -connect <nas_host>:<configured_port> < /dev/null | awk '/BEGIN/,/END/{ if(/BEGIN/){a++}; out="cert"a".pem"; print >out}' 
```

After you provided the certificate to the Docker daemon, you can again execute the command below on your NAS.

```bash
docker pull <nas_ip_or_host>:<configured_port>/doesnotexistyet:latest
```

You should now see the already mentioned error message `Error response from daemon: manifest for <nas_host>:<configured_port>/doesnotexistyet:latest not found: manifest unknown: manifest unknown`.

You will have to do the same or similar steps on every machine from which you want to access the Docker registry, e.g. to push a new image.
This depends on the operating system you use.

* In case of Linux, the approach already described for the NAS is valid.
Only the directory to put the certificate to is different.
Normally it is `/etc/docker/certs.d/<nas_ip_or_host>:<configured_port>` on Linux.

* According to the Docker [documentation][cert-mac] related to Docker Desktop for Mac it is sufficient to add the certificate of the certification authority to the Mac's keychain.

* The [documentation][cert-win] related to Docker Desktop for Windows describes a similar approach.
Simply add the certificate of the certification authority to the Windows certificate store.
You have to put it under the `Trust Root Certification Authorities` or `Intermediate Certification Authorities` and Docker will pick it up.

# Push an Image

Due to the fact that you read this post which describe how to run a private Docker registry, I assume that you are familiar with Docker and that you know you to build and push an image to a registry.

I recommend to push an image now, which we can use to create a container using the Synology user interface.

# Configure the Docker Registry on the NAS

Since we now have a working Docker registry available, we can start to use it.
First of all it needs to be registered.
To do this, open the Docker app in the Synology user interface, navigate to the registry section and click on settings.
There you can add a new registry.
Make sure to enter the data according to your setup.

![Add a new Docker Registry](/img/docker_registry_add.png){: .center-image }

Once your registry is added, you can select it and mark it as the one in use.
Please keep in mind that you can have only one registry active.
From now on the NAS uses your new registry to search for images.

[oh-docker-fabian]: https://www.fabian-keller.de/blog/running-openhab-on-a-qnap-nas-with-docker/
[docker-hub]: https://hub.docker.com/
[docker-hub-pricing]: https://hub.docker.com/pricing
[docker-registry-docs]: https://docs.docker.com/registry/
[docker-registry-image]: https://hub.docker.com/_/registry
[docker-insecure]: https://docs.docker.com/registry/insecure/
[docker-certs-docs]: https://docs.docker.com/engine/security/certificates/
[synology-certs]: https://www.synology.com/en-us/knowledgebase/DSM/help/DSM/AdminCenter/connection_certificate
[openssl-stackexchange]: https://unix.stackexchange.com/questions/368123/how-to-extract-the-root-ca-and-subordinate-ca-from-a-certificate-chain-in-linux
[cert-mac]: https://docs.docker.com/docker-for-mac/#add-custom-ca-certificates-server-side
[cert-win]: https://docs.docker.com/v17.09/docker-for-windows/faqs/#how-do-i-add-custom-ca-certificates