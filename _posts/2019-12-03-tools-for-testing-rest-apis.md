---
layout: post
title:  "Tools for Testing REST APIs"
category: IT
tags: tools api
bigimg: /img/bigimg_tools-for-testing-rest-apis.jpg
credname: Lachlan Donald
credurl: https://unsplash.com/photos/YVT5aF2QM7M
comments: true
origin: https://blog.mimacom.com/tools-for-testing-rest-apis/
excerpt: A regular task in the life of a software developer is to choose appropriate tools for specific use cases. That's why I would like to have a short look at several tools which I already used for testing REST APIs. To be more concrete, it's about cURL, JMeter, SOAP UI, Swagger UI, Postman and IntelliJ Idea.
---

A regular task in the life of a software developer is to choose appropriate tools for specific use cases.
That's why I would like to have a short look at several tools which I already used for testing REST APIs.
To be more concrete, it's about cURL, JMeter, SOAP UI, Swagger UI, Postman and IntelliJ Idea.

I will describe these tools in the following sections briefly.
This article is explicitly not a tool comparison which lists all features and compares them in detail.
So please consider this as my personal opinion.

## cURL

A very basic commandline tool for sending HTTP requests is [cURL][curl], which is part of many Linux distributions as well as MacOS and even Windows 10.

It can be used for ad-hoc requests sent manually from the commandline as well as for script based automation of sending HTTP requests, parsing the response and evaluating the result.

![Using cURL to Send a Request to a REST API](/img/rest_curl.png)

cURL is a very convenient tool for simple use cases like sending a single ad-hoc request to a REST API.
Compared to other tools, it lacks usual features.
Therefore it's good to know that it's there, but I don't use it too often.

## JMeter

In the area of load tests, JUnit is a well known tool.
It can also be used to send ad-hoc requests, but its main strength is related to automated load tests.
The tool provides a GUI as well as a CLI mode.
Usually, the GUI is used to define the tests, which can then run as a load test via CLI.
For directly sending some parametrized requests to a REST API, the GUI is pretty sufficient.

JMeter offers the possibility to group requests, to extract values from responses and to perform checks.

![Using JMeter to Send a Request to a REST API](/img/rest_jmeter.png)

I have to admit that JMeter is mighty but hard to learn.
In the past, I used it a lot.
If you use it regularly, you can do pretty cool things with it - but if you don't use it for a longer time, you can start to learn it again.
At least this is my impression while trying to send some requests as a demo for this blog post.

## SOAP UI

A tool which has quite a history is [SOAP UI][soapui].
Besides sending SOAP requests, it also offers the possibility to send requests to REST APIs.
Another interesting feature is to mock APIs for local testing.

SOAP UI is available in an open source edition as well as in a paid pro version.

![Using SOAP UI to Send a Request to a REST API](/img/rest_soapui.png)

The fact that SOAP UI provides support for SOAP APIs as well as for REST APIs is a good argument if you have to deal with both types of API in your daily work.

## Swagger UI

Luckily it's pretty usual to provide Open API specifications for REST APIs.
If there is one for the API you want to test, it's not a big deal to import it into [Swagger UI][swagger-ui] to get a dynamic REST API client.
It is possible to send the request directly from within the documentation.
The Swagger UI provides the corresponding cURL command, which can be used in the commandline.

You can use the one which comes with the online [Swagger Editor][swagger-editor] or you can use your own one either locally or deploy it to one of your servers.

![Using Swagger UI to Send a Request to a REST API](/img/rest_swaggerui.png)

For me, this is quite often the starting point, because many APIs already provide a Swagger UI.
If this is the case, it is not a big deal to click some buttons and to fill in the necessary request parameters.

## Postman

I think [Postman][postman] is a pretty well known REST client, which is used a lot.
It can be used to define parametrized request, to extract values from the response and to orchestrate several requests.

One cool feature is that workspaces, e.g. for different projects, can be defined.
It is also possible to share such workspaces within a team using a Postman account.
This team share feature is useful.
But you should be aware, that you should think about how you define your credentials.
Otherwise, they are also synced to the cloud, which is maybe not what you want.

![Using Postman to Send a Request to a REST API](/img/rest_postman.png)

Postman is the tool I switched to after my career as JMeter power user.
It's very convenient and provides lots of features, which help when working with REST APIs.

## IntelliJ Idea

IntelliJ Idea offers a [built-in HTTP client][intellij-idea], which operates with input from simple text files.
It is possible to define variables, which are managed in a separate file.
Besides the possibility to manage those request files also in Git, this is for sure one major strength of this tool. 

A nice feature is, that it is possible to paste a cURL requests to such a file which is then converted to the corresponding IntelliJ Idea request definition on the fly.

![Using IntelliJ Idea to Send a Request to a REST API](/img/rest_intellij-idea.png)

Since IntelliJ Idea is my favorite IDE, I really love the ability to also test REST APIs without switching to another tool.
IntelliJ Idea is the tool I currently use the most for interacting with REST APIs to test them.

## My Personal Preferences

During the short description of the tools, I already gave a brief note about my opinion.
To consolidate this, I would recommend to try out Postman and the features provided by IntelliJ Idea.

The other tools also do a great job, but they aim at slightly different use cases.

## Conclusion

In this blog post, I presented some of the tools for REST API testing, which are in my toolbox.
This article was a pretty short overview which is for sure not complete in terms of features of the tools, and of course, there are many other tools for this purpose out on the market.

I'm always happy to hear which tools other people use.
So feel free to try out the ones I mentioned and let me know if there are other tools which are worth a try in the comment section.

[curl]: https://curl.haxx.se/
[jmeter]: https://jmeter.apache.org/
[soapui]: https://www.soapui.org/
[swagger-ui]: https://swagger.io/tools/swagger-ui/
[swagger-editor]: https://editor.swagger.io/
[postman]: https://www.getpostman.com/
[intellij-idea]: https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html