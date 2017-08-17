---
layout: post
title:  "Use Jenkins as scheduler"
category: IT
tags: jenkins administration
comments: true
---

When talking about a simple and lightweight scheduling solution in a system administration context you might think about something like [cron][cron-def]. I used it for a long time starting with my first home server.

This was easy enough to maintain for a single machine. But as time passed by and new machines like a NAS system and a Raspberry Pi for home automation purposes, this got too complicated from my point of view. The main downside is the maintenance of the jobs on each machine and the overall scheduling, i.e. the scheduling of jobs and management of job dependencies between the machines.

The concrete scenario is, that my home server as well as the NAS should be shutdown each night but not before the backup of home automation data copied from the Raspberry Pi to the NAS finished.

## General setup 

Coming from this use case, I stumbled across a Jenkins managed solution. In fact it is really simple. 

* Jenkins runs on a single machine.
* Central Jenkins jobs are created instead of cron jobs on each system.
* Those Jenkins jobs connect to the corresponding system via SSH and execute the command locally.

## Prepare the Jenkins server
If not already done, a key pair needs to be generated for the user Jenkins runs with. Using the default settings, the public key file will be located in `~/.ssh/id_rsa.pub` after the key pair was generated. You'll need this public key later on for setting up the access to the target machines.

{% highlight console %}
jenkins@srv01:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jenkins/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/jenkins/.ssh/id_rsa.
Your public key has been saved in /home/jenkins/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:V6VruYNPe33n1TtDRkeNq9Kg3Etu4kjkaTpsy3EdNw4 jenkins@srv01
The key's randomart image is:
+---[RSA 2048]----+
|              ...|
|             o. o|
|            o  o |
|          .. o...|
|      ..Eo+o+.. .|
|     o oo*++o. o.|
|   .. * .ooo+ o.o|
|   .+* .. +o o.o*|
|   .+o...o  o. o*|
+----[SHA256]-----+
jenkins@srv01:~$ cat /home/jenkins/.ssh/id_rsa.pub
ssh-rsa AAAAB3Nz...LqFpI3 jenkins@srv01
{% endhighlight %}

## Prepare the target machine

{% highlight console %}
ahirsch@srv02:~$ sudo useradd -m scheduler
ahirsch@srv02:~$ sudo su scheduler
scheduler@srv02:/home/ahirsch$ cd ~
scheduler@srv02:~$ mkdir .ssh
scheduler@srv02:~$ echo ssh-rsa AAAAB3Nz...LqFpI3 jenkins@srv01 > .ssh/authorized_keys
{% endhighlight %}

[cron-def]: https://en.wikipedia.org/wiki/Cron
