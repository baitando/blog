---
layout: post
title:  "Azure Functions Implemented with Spring Cloud Function"
category: IT
tags:
    - azure
    - cloud
    - java
    - java
    - serverless
    - spring
bigimg: /img/bigimg_try.jpeg
credname: Thirdman
credurl: https://www.pexels.com/de-de/foto/text-5981701/
comments: true
origin: https://blog.mimacom.com/azure-functions-implemented-with-spring-cloud-function/
excerpt: On my serverless journey, I had a closer look at the technologies around. Since I have been implementing applications with Spring, it was natural to try Spring Cloud Function. Let's have a closer look at how a serverless function running as Azure Function can be implemented with Spring.
---

On my serverless journey, I had a closer look at the technologies around.
Since I have been implementing applications with Spring for many years, it was natural to try Spring Cloud Function.
Let's have a closer look at how a serverless function running as Azure Function can be implemented with Spring.

In one of my last [blog posts][post-azure-function-java], I had a look at how a serverless function implemented in Java looks like, when it makes use of the Azure Function offering.
If you want to have a look at this plain Java Azure Function first, you can find the blog post [here][post-azure-function-java].

Similar to the last blog post, we will implement an HTTP endpoint.
We pass in a name as query parameter and receive a greeting message in the response.
The source code of the complete sample is available on [GitHub][sample-github].

## Motivation for Trying Out Spring Cloud Function

In addition to reducing the coupling of my serverless function to Azure specifics, I am interested in making use of the known Spring Framework and Spring Boot features like dependency injection, the external configuration hierarchy and similar.

Although knowing that Spring bootstrapping could be a bit heavy for a serverless function, I decided to give it a try to gain some more practical insights and to evaluate, if I can benefit from the goals advertised on the [Spring Cloud Function website][spring-cloud-function-website] (this is just an excerpt):

* Decoupling of specific logic from runtime targets.
* Consistent programming model across cloud providers.
* Possibility to use known Spring Framework and Spring Boot mechanisms.

## How Spring Cloud Function Integrates with Azure Function

To get started, there is some Maven related configuration necessary.
For this, I would like to link to the [pom.xml file of the sample][sample-pom].

Using Spring Cloud Function with Azure Function, we implement the Azure Function specific class acting as the entry point.
This method is annotated with the Azure Function annotation `FunctionName`.
In addition, it defines the Azure Function trigger, which is a `HttpTrigger` here.

The difference with Spring Cloud Function is, that this class now extends the `FunctionInvoker` class provided by Spring.
To be more precise, this class is provided by the Spring Cloud Function adapter for Azure Functions.
Similar adapters exist for AWS Lambda and Apache OpenWhisk.

```java
public class GreetingHandler extends FunctionInvoker<String, String> {

    @FunctionName("greetings")
    public String execute(
            @HttpTrigger(
                    name = "req",
                    methods = {HttpMethod.GET},
                    authLevel = AuthorizationLevel.ANONYMOUS
            ) HttpRequestMessage<Optional<String>> request,
            ExecutionContext context) {
        context.getLogger().info("Processing a greeting request.");
        final String name = request.getQueryParameters().get("name");

        return handleRequest(name, context);
    }
}
```

The `FunctionInvoker` is the main player related to the integration.
It provides the `handleRequest` method, which is called from within the triggered Azure Function method.
From that point on, the `FunctionInvoker` takes care of bootstrapping the Spring Boot application, processing the request and returning the response.

One thing to keep in mind is, that the Spring Boot application starts as part of processing within the `handleRequest` method.
Therefore, you won't be able to e.g. use dependency injection or the Spring Boot logging mechanism in this adapter class.

Please note, that the integration changed some time ago.
Prior to the `FunctionInvoker`, the now deprecated `SpringBootRequestHandler` was used.

## Implementing the Logic

The main part of the serverless function is related to the business logic, which should be independent of the runtime it runs on and also independent of the way, how the logic is consumed (e.g. via HTTP).
To achieve this, we make use of the Java `Function` interface.
The `GreetingFunction` class holding the business logic implements this interface and provides the respective `apply` method.

The business logic in fact is fairly simple here.
We only build the welcome message and return it.

```java
public class GreetingFunction implements Function<String, String> {

    @Override
    public String apply(String name) {
        return "Welcome, " + name;
    }
}
```

## Connect the Azure Function to the Logic

Now that we have the infrastructure part for the integration with Azure Function and the part implementing the business logic, it is time to wire things together.
Since we use Spring Cloud Function, we provide a Spring Boot application class similar to any other Spring Boot application we implement.

```java
@SpringBootApplication
public class GreetingApplication {

    public static void main(String[] args) {
        SpringApplication.run(GreetingApplication.class, args);
    }

    @Bean
    public GreetingFunction hello() {
        return new GreetingFunction();
    }
}
```

To keep out Spring specific annotations like `@Component` from the class holding the business logic, we implement a `@Bean` annotated method in the `SpringBootApplication` class.
This method creates an instance of the function and returns it to make it available in the Spring application context.

## Necessary Configuration for Running the Serverless Function

For starting the Spring Boot application, it is necessary to specify the main class, i.e. the class annotated with `@SpringBootApplication`.
This setting is provided as environment variable `MAIN_CLASS` when running on Azure in the cloud.
When running the Azure Function locally, we can provide such environment variables in the `local.settings.json` file.
Having a look at the sample code on GitHub, you find the following content in the `src/main/azure/local.settings.json` file:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "java",
    "MAIN_CLASS":"de.baitando.azure.samples.springfunction.GreetingApplication"
  }
}
```

This file is automatically used, when the `pom.xml` file is configured like in the sample and when you run the Azure Function locally using the Maven plugin.
If you run the Azure Function e.g. using the built-in feature of Intellij Idea, pay attention to specify the same settings in the run configuration settings.
An example reflecting the content of the `local.settings.json` file from above is shown in the screenshot below.

![Run configuration for running the Azure Function from Intellij Idea](/img/spring-function_run-configuration-intellij.png)

If you forget to specify the `MAIN_CLASS` you will likely have a hard time to analyze the root cause.
In this case, the Azure Function will start successfully, but processing the request will fail with a `NullPointerException`.
This occurs, because there is no function instance (function in the sense of the Java `Function` interface, not in the sense of Azure Function) registered.

```
[2021-10-31T14:41:55.959Z] Caused by: java.lang.NullPointerException
[2021-10-31T14:41:55.959Z] 	at org.springframework.cloud.function.adapter.azure.FunctionInvoker.handleRequest(FunctionInvoker.java:120)
[2021-10-31T14:41:55.959Z] 	at de.baitando.azure.samples.springfunction.GreetingHandler.execute(GreetingHandler.java:26)
[2021-10-31T14:41:55.959Z] 	... 17 more
[2021-10-31T14:41:55.959Z] .
```

## Test if it Works

To test if it works, we start the Azure Function runtime locally.
You can do this either by running the Azure Function Maven plugin or by using a properly configured run configuration in your IDE.

Once the Azure Function is running, you can send a HTTP GET request to [http://localhost:7071/api/greetings?name=mimacom%20Blog%20Visitor](http://localhost:7071/api/greetings?name=mimacom%20Blog%20Visitor).
If everything works fine, you should receive HTTP 200 OK with the message `Welcome, mimacom Blog Visitior`.

An Intellij request file is provided in the `requests/rest.rest` file of the [GitHub repository][sample-github].
Sending the request, you should receive a response similar to the one shown below.

```
Date: Sun, 31 Oct 2021 15:38:16 GMT
Content-Type: text/plain; charset=utf-8
Server: Kestrel
Transfer-Encoding: chunked

Welcome, mimacom Blog Visitor

Response code: 200 (OK); Time: 223ms; Content length: 29 bytes
```

## How Everything Works Together

We have seen, how the different parts need to be implemented and configured.
Now let's have a brief look at how the parts conceptually work together.
We assume, that the Azure Function is running and that the runtime, on which the implemented Azure Function is hosted, listens (e.g. the local servlet container).

1. A consumer sends an HTTP GET request to [http://localhost:7071/api/greetings?name=mimacom%20Blog%20Visitor](http://localhost:7071/api/greetings?name=mimacom%20Blog%20Visitor).
2. The request is routed to the `execute` method of the `GreetingHandler` class.
3. Processing the request is delegated to the `handleRequest` method of the parent `FunctionInvoker` class.
4. The `FunctionInvoker` starts the Spring Boot application, which is specified in the `MAIN_CLASS` environment variable. It searches for a function registered in the function catalog. It finds the instance of the `GreetingFunction` and triggers the `apply` method.
6. The method `GreetingFunction` builds the welcome message and returns it.
7. The return value is handed back to the `GreetingHandler` and is in the end passed to the execution context, which takes care of returning a proper HTTP response.

## Conclusion

In this blog post, we had a closer look at how an Azure Function can be implemented with Spring Cloud function to allow making use of Spring Boot mechanisms as well as for decoupling Azure Function specific runtime configuration from the core business logic.

Please note, that this blog post only describes how to implement an Azure Function in Java using Spring Cloud Function.
This does not mean, that this is a good idea in specific use cases.
Especially when it comes to cold starts, the startup time related to Java and especially Spring Boot can be a showstopper.
There are some tweaks related Spring Cloud Function as well as ideas to compile to native - but such topics are big enough for separate blog posts.

If you want to try out the sample used here, you can find the source code on [GitHub][sample-github].
Feel free to have a look and try it out yourselves.

[post-azure-function-java]: https://blog.mimacom.com/implementing-azure-functions-with-java/
[spring-cloud-function-website]: https://spring.io/projects/spring-cloud-function
[sample-github]: https://github.com/baitando/azure-samples/tree/master/spring-function
[sample-pom]: https://github.com/baitando/azure-samples/blob/master/spring-function/pom.xml
