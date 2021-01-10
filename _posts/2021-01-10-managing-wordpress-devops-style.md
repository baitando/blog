---
layout: post
title:  "Managing Wordpress DevOps Style"
category: IT
tags: wordpress devops
bigimg: /img/bigimg_bright-spot.jpg
credname: Annebell Dogger
credurl: https://unsplash.com/photos/tVJKaxFYEk4
comments: true
excerpt: Wordpress is widely used, which has a reason. It is easy to use and cheap to host. On the other side, it is somehow fragile with all its plugins. That's why I decided to apply well known DevOps practices to the Wordpress websites I manage to reduce risk.
---

Many people like Wordpress to run websites, because it seems to be easily usable and manageable at first glance.
Wordpress uses the traditional stack consisting of an Apache webserver, PHP, and a MySQL database, which is available for cheap money.
Some hosting providers even provide one-click installers or managed Wordpress websites, which further reduce operations effort at a decent price.

Nevertheless, problems begin with the first update - at least from my experience.
It is easy to click the update button in the admin section, but it is hard to figure out problems that arise from incompatible versions.
I therefore always recommend trying it out on a non-production environment first and to have proven backup and restore mechanisms in place.

When the number of Wordpress websites I technically manage grew, I decided to apply the mechanisms I use in my regular job as software engineer in order to increase efficiency.
Namely, the goal was to manage Wordpress websites in a version controlled repository, to put proper dependency management in place, and to provide scripts for automation of build, deployment and various other tasks.

In this blog post I would like to give an overview of the approach I use to manage Wordpress websites.
You can find a working example of the scripts in [this GitHub repository][github-samples].

# Goal
The main goal was to manage websites similar to software in software engineering.
Every version of the website should be reproducible from a version controlled repository.
A version in this context includes the Wordpress files, but not the content (means database and uploaded files) itself.

With regard to dependency management, the following features are needed:
* Manage the Wordpress core version as well as the versions of all additional plugins and themes in a declarative way, e.g. in a file like Java developers know from the Maven `pom.xml` file.
* Resolve publicly available dependencies from public dependency repositories instead of managing their source in the website code repository.
* Allow adding plugins and themes which are not publicly available, i.e. which are either custom implementations or paid ones and therefore not available in public dependency repositories.

To minimize the manual effort related to deployments and distribution to multiple stages (production and test in my case), scripts for automating the following tasks are necessary:
* Initialize a new website on a remote server according to the website definition and optionally secure it with HTTP basic auth.
* Export and import data of an existing Wordpress website for cloning environments and for the sake of backup and recovery.
* Create a local Wordpress website based on Docker and initialize it with exported data of an existing Wordpress website.

# Prerequisites
The solution I implemented is based on several prerequisites, which must be fulfilled in order to use it.
It is necessary to have remote access to the remote server via SSH.
The user which connects to the target server must be eligible to use key-based authentication.

Additionally, several commands must be available on the remote server.
These are the following ones:

* `php` for executing PHP scripts like the `wp-cli`.
* `mysqldump` for exporting a dump of the MySQL database.
* `mysql` for importing a dump of the MySQL database.
* `htpasswd` for securing the website with HTTP basic authentication.

For being able to create and use a local copy of the website, you need `docker` and `docker-sync`.

# Dependency Management
[Composer][composer] is a widely used dependency manager for PHP, which makes it an obvious choice to manage the website dependencies.
The file containing the dependency declarations is called `composer.json`.

The sample uses the Wordpress distribution provided by [johnpbloch][wordpress-package] in the [Packagist][wpackagist] dependency repository.
To resolve the Wordpress plugins and themes, the [WordPress Packagist][wpackagist] dependency repository must be added.

# Custom Code And Paid Dependencies
Custom code and paid plugins have on thing in common: They are both usually not available in public dependency repositories.
It could be possible to resolve paid dependencies from private repositories, but if you would not like to provide a custom, private repository for your own code, this point would still not be solved.

I therefore decided to treat paid dependencies similar to code I implement myself:

* In case of custom implementations, I provide the code in the `src` directory of my website repository.
* If paid dependencies exist, I download them and put them also to the `src` directory.

Please be aware that you may not be allowed to pubish paid dependencies in public code repositories, if you ever would publish your website sources publicly.

# Build
To create the deployable package of the website, Composer is triggered to resolve the dependencies.
The result of this operation consists of the Wordpress core and publicly available plugins and themes.

In the next step, the paid and custom plugins and themes are added.
As described earlier, the source code of both is available in the code repository of the website.
Therefore, these sources are copied to the Composer output directory to complete the deployment package.

In the scripts provided in the [sample repository][github-samples], the build is triggered with the `./baitando build` command.

# Deployment
The deployment is based on the build operation, which needs to be executed in advance.

The Wordpress files are deleted from the server, but the data consisting of the database, and the uploaded user files (located in the `wp-content/uploads` directory) remain there.
Afterwards, the build result is uploaded to the server to replace the old Wordpress files.

In the scripts provided in the [sample repository][github-samples], the deployment is triggered with the `./baitando deploy <env>` command.

# Initialization
During initialization, the `wp-config.php` file containing the Wordpress configuration is generated by using the `wp-cli` tool provided by Wordpress.
Since the `wp-cli` requires a valid Wordpress installation, you need to deploy the website before the environment is initialized.

If the website should be secured with HTTP basic authentication, a proper `.htpasswd` file is generated, and the necessary configuration is appended to the `.htaccess` file of the website.

In the scripts provided in the [sample repository][github-samples], the initialization is triggered with the `./baitando init <env>` command.

# Export
The export takes care to dump the database and to copy all the data files of the website.
The result is then downloaded to the computer, which runs the script.

This can either be used for backups, or for cloning environments.

In the scripts provided in the [sample repository][github-samples], the export is triggered with the `./baitando export <env>` command.

# Import
The import takes care to import a database dump and to replace the data files with the data files of an export.

This can either be used to restore an environment, or for cloning environments.

In the scripts provided in the [sample repository][github-samples], the export is triggered with the `./baitando import <env>` command.

# Local Environment
In addition to the previously explained operations, there is the part of creating and using a local, Docker-based version of the website for development purposes.

The approach is based on Docker Compose, which allows defining and connecting a set of containers.
For a Wordpress website, the essential setup consists of an Apache webserver and a MySQL database.

A `docker-compose.yaml` file defines both containers.
The MySQL database is initialized with a dump file by simply mounting a directory of the host computer, which contains an SQL dump.
The SQL dump will be picked up during container initialization to load the initial data.
The Wordpress files delivered by the webserver are located in a directory of the host computer, which gets mounted into the webserver container.

When I tested this, the result was horribly slow on my MacBook.
This was due some known Docker issues with the filesystems on Macs, which also affects Windows, but not Linux.

That's why I decided to make use of [docker-sync][docker-sync], which speeds up filesystem operations on Macs.
The downside of this is, that you need another tool and technology (`docker-sync` requires Ruby) to make everything work - but without such performance tuning you won't be able to effectively use the local environment.
Please have a closer look at the documentation of [docker-sync][docker-sync] to get more insights about how it works and how it is used.

The scripts in the [sample repository][github-samples] are structured in a way, that it is required to build the website and to export the data of an existing environment before creating the local environment.
The result is then used during `./baitando docker init` to create the Docker images, to initialize `docker-sync`, to create the webserver and database containers, and to generate the initial configuration using a temporary `wp-cli` Docker container.

Once the initialization of the local environment is done, it can be started with `./baitando docker start` and stopped with `./baitando docker stop`.
To completely delete the local environment, simply run `./baitando docker destroy`.

# Limitations
The solution at the time this blog post was written had several limitations, which could be solved by further enhancing the scripts.

The most relevant limitations are likely:

* The assumption is, that it is possible to access the server via SSH to run commands remotely.
This is often not possible, especially when cheap hosting offerings are used.
* Using a local environment based on Docker is currently limited in the way, that you can create a local copy of the website - but there is no development workflow, which makes changes in custom sources available to the Docker-based environment.

# Conclusion
Wordpress is widely used for different kinds of websites, and it looks like it does a good job.
My personal impression until now was, that it is cumbersome to deal with backup and restore processes and that even updates of external plugins and themes are really cumbersome without a fast, tested way back to the previous state.

Spending some effort in building proper scripts to automate everything, I recognized that it is still not nice to operate a Wordpress website - but I now feel a bit more confident in running usual operations on production websites.
It is now possible to simply create a clone of a website (without using any Wordpress plugins to clone the website) on another server or even locally to test the impact of plugin and theme updates.

I also remember a situation in which a website I took over was infected with malware a long time ago.
To fix this, I had to completely set up the website from scratch - including manually downloading Wordpress and all plugins.
This would have been way easier if there had been a declarative dependency management approach similar to the one presented here.
I now feel better prepared.

Feel free to adapt the scripts the way they help you.
Please be aware, that they could be very specific to my use cases at some points.
It is also important to know, that there were no tests for other use cases as mine.
Be careful, take backups before you use the scripts, and ideally have a look at the scripts yourself before applying them.

[github-samples]: https://github.com/baitando/wordpress-samples
[composer]: https://getcomposer.org
[wordpress-package]: https://packagist.org/packages/johnpbloch/wordpress
[wpackagist]: https://wpackagist.org
[docker-sync]: http://docker-sync.io