---
layout: post
title:  "Setting Up A Raspberry Pi Kubernetes Cluster"
category: IT
tags: kubernetes
bigimg: /img/bigimg_container_management.jpg
credname: chuttersnap
credurl: https://unsplash.com/photos/kyCNGGKCvyw
comments: true
excerpt: As an education project, I decided to set up a playground Kubernetes cluster with a bunch of Raspberry Pis. Let's see how this works with the Kubernetes compatible container platform k3s.
---

I really like the Raspberry Pi and use lots of them for different use cases.
In this blog post, I would like to explain, how I built a Kubernetes cluster with some Raspberry Pis from scratch.

With this project, I wanted to educate myself in how Kubernetes works, how it is set up with real hardware and how it is operated.
I hope this will give some useful knowledge, also in Cloud scenarios where you often rely on managed Kubernetes clusters.

# Network Infrastructure

I decided to have three nodes in the cluster including the master.
To keep the cluster hardware separated from the rest, a separate switch is used.
All the nodes are connected to the switch and the switch is connected to the internet router.

The benefit of a dedicated switch is, that all cluster related network cables are connected to the same switch while all nodes remain directly accessible in the existing network.

TODO: Diagram or image

# Shopping Tour

First it is necessary to obtain the necessary hardware.
My setup consists of

* 3x Raspberry Pi 4 Model B with 4 GB RAM
* 3x SanDisk Extreme microSDHC cards with 32 GB storage
* 1x MakerFun Raspberry Pi rack incl. fans
* 4x deleyCon patch cables with a length of 0,5 m
* 3x Syncwire USB C to USB 3.0 cable
* 1x Anker PowerPort USB power station with 60 W and 10 USB ports
* 1x Netgear GS108E Gigabit ethernet switch with 8 ports

There were bundles offered, which contained more parts than needed - e.g. the USB cables were available in packages of two.
In addition, I was not sure about the length of the ethernet calbes.
That's why I bought cables of 0,5 m length as well as cables of 0,25 m length.

At the time I bought the hardware, the total price of this basket was approx. 270 €.
Please find a picture below.

![Hardware parts needed for the cluster](/img/cluster_parts.jpeg)

# Assembling

The most time-consuming part is assembling the rack, the fans, and the Raspberry Pis.
I recommend to start from bottom to top.


 