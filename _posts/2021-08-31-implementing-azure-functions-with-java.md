---
layout: post
title:  "Implementing Azure Functions with Java"
category: IT
tags: azure java serverless
bigimg: /img/bigimg_functions.jpeg
credname: David Iskander
credurl: https://unsplash.com/photos/iWTamkU5kiI
comments: true
origin: https://blog.mimacom.com/implementing-azure-functions-with-java/
excerpt: Azure Functions support several runtime environments including the Java Virtual Machine, which allows us to implement functions in programming languages like Java and Kotlin. Let's have a look at how a simple Azure Function can be implemented with Java.
---

Azure Functions support several runtime environments including the Java Virtual Machine, which allows us to implement
functions in programming languages like Java and Kotlin.
Let's have a look at how a simple Azure Function can be implemented with Java.

The sample Azure Function introduced here is available on [GitHub][sample-plain].
Feel free to have a look at it.

## Characteristics of an Azure Function

There are several ways to create the basic skeleton of an Azure Function project implemented with Java and built with Maven.
In the [Microsoft tutorial][microsoft-tutorial], the project is manually created from scratch by creating the
folder structure, copying a `pom.xml` template, and implementing the necessary classes.
For a quickstart, I recommend  using the alternative approach of using the built-in wizard of tools like Intellij Idea.

An Azure Function project basically consist of:

* The `host.json` file. This file configures the Azure Function runtime and is applied locally as well as in the cloud.
* The `local.settings.json`. This file locally defines settings applied to the function. When running in the cloud, such
  settings are defined via app settings.
* A `pom.xml` file. This file defines the necessary dependencies and plugins, which e.g. take care of packaging or
  executing the function on the local machine during development.
* The handler class. This class defines the execution trigger and is where you implement the function logic to run.

## Azure Function Runtime Configuration

The `host.json` file defines some global settings which affect the runtime in which the function is executed.
In case of the sample illustrated here, the file looks as shown below.
A complete description of this file and the available settings is available in the [documentation][host-file].

```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[1.*, 2.0.0)"
  }
}
```

The `version` setting specifies the schema version of the `host.json` file. If using the Azure Function runtime `v2`
or `v3`, which are the two latest versions available at the moment.

The `extensionBundle` setting is used to specify function binding extensions required by the Azure Function.
Such extension bundles define the available Azure Function triggers.

## Local Function App Settings

The `local.settings.json` file is used to provide environment variables when running the Azure Function locally.
There are several settings like the web server port, which are only applied to the local Azure Function runtime (see a full list [here][settings-file-local]).

It is possible to define custom variables, which are provided at runtime and which are accessible by the custom implementation of your Azure Function.
When deployed to the cloud, such configuration is provided using the app settings mechanism, which allows e.g. to securely reference secrets from an Azure Key Vault.
Confidential information like connection strings, usernames or passwords provided this way, are the reason why you must exclude such a local settings file from version control, whenever you access a real system like a database in the cloud or similar.
In case of a local e.g. Docker Compose based infrastructure, you can safely manage the `local.settings.json` file in your version control system, as long as there are only values from this local infrastructure contained.

In addition, there are some Azure Function environment variables, which are applied to the local as well as to the cloud environment (see a full list [here][settings-file-global].
Such configuration can be specified in the `local.settings.json` file for the local environment and using the Function App settings used in the cloud.

## Maven Project Definition

The `pom.xml` file defines the dependencies and the build process of the Maven project. In case of an Azure Function, we make use of the Java library provided by Microsoft.
It provides us with the API classes, which are used to integrate with the Azure Function runtime. The dependency looks like shown below.

```xml

<dependency>
    <groupId>com.microsoft.azure.functions</groupId>
    <artifactId>azure-functions-java-library</artifactId>
    <version>${azure.functions.java.library.version}</version>
</dependency>   
```

To create the Azure Function package during the build process, to make it possible to run the function locally, and to deploy the result to the Azure environment, the Azure Function Maven plugin is included.
It requires some settings, which define the deployment specifics like the function app name, the resource group, the app service plan, the region, and the runtime operating system.
If one of those resources does not exist, the plugin takes care of creating it.

```xml

<plugin>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-functions-maven-plugin</artifactId>
    <version>${azure.functions.maven.plugin.version}</version>
    <configuration>
        <appName>${functionAppName}</appName>
        <resourceGroup>${resourceGroup}</resourceGroup>
        <appServicePlanName>${appPlan}</appServicePlanName>
        <region>${region}</region>
        <runtime>
            <os>${os}</os>
            <javaVersion>${javaVersion}</javaVersion>
        </runtime>
        <appSettings>
            <property>
                <name>FUNCTIONS_EXTENSION_VERSION</name>
                <value>~3</value>
            </property>
        </appSettings>
    </configuration>
    <executions>
        <execution>
            <id>package-functions</id>
            <goals>
                <goal>package</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

The deployment package is a Zip file created during the Maven `package` phase.
Unfortunately this plugin does not cover everything needed.
The Azure Function plugin requires to first provide additional resource files and dependencies in a specific folder beforehand.

To provide additional resources, we make use of the Maven Resources plugin.
It will copy the `host.json` and `local.settings.json` files from the source folder to the folder used by the Azure Function plugin.

```xml

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.1.0</version>
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>package</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <overwrite>true</overwrite>
                <outputDirectory>${stagingDirectory}</outputDirectory>
                <resources>
                    <resource>
                        <directory>${project.basedir}</directory>
                        <includes>
                            <include>host.json</include>
                            <include>local.settings.json</include>
                        </includes>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

The Maven Dependency plugin is used to provide the dependency jar files in the folder used by the Azure Function plugin.
The only dependency excluded for the deployment package is the Azure Function library we earlier defined.

```xml

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.1</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${stagingDirectory}/lib</outputDirectory>
                <overWriteReleases>false</overWriteReleases>
                <overWriteSnapshots>false</overWriteSnapshots>
                <overWriteIfNewer>true</overWriteIfNewer>
                <includeScope>runtime</includeScope>
                <excludeArtifactIds>azure-functions-java-library</excludeArtifactIds>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Due to the fact, that the local executing of the Azure Function during  development creates a .NET specific folder containing some artifacts required by the Azure Function runtime, the Maven  Clean plugin is configured to delete that folder during the Maven `clean` phase.

```xml

<plugin>
    <artifactId>maven-clean-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <filesets>
            <fileset>
                <directory>obj</directory>
            </fileset>
        </filesets>
    </configuration>
</plugin>
```

## Function Implementation

Now that the Azure Function specific project setup is done, we can start implementation.
For demonstration purposes, the sample exposes and HTTP endpoint which receives a name as query parameter and returns a greeting string.
To achieve this, a new Java class is created.
The entrypoint of the Azure Function is a method annotated with `@FunctionName`.

This method defines the trigger as annotation within the parameter list.
There are several triggers available, and it is possible to make use of additional ones using extensions specified in the `host.json` file.
In case of an HTTP endpoint, the trigger used is the `@HttpTrigger`.
The annotation is placed on the method parameter, which is in this case of type `HttpRequestMessage`.
Additional configuration like the request method can be configured as parameters of this annotation.

```java
public class GreetingFunction {

    @FunctionName("greetings")
    public HttpResponseMessage run(
            @HttpTrigger(
                    name = "req",
                    methods = {HttpMethod.GET},
                    authLevel = AuthorizationLevel.ANONYMOUS)
                    HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        context.getLogger().info("Processing a greeting request.");

        final String name = request.getQueryParameters().get("name");

        if (name == null) {
            return request.createResponseBuilder(HttpStatus.BAD_REQUEST).body("Please pass a name on the query string").build();
        } else {
            return request.createResponseBuilder(HttpStatus.OK).body("Hello " + name).build();
        }
    }
}
```

Inside the method we can use the `request` parameter to get the name, which is passed as query parameter `name`.
This value is used to construct a greeting message which is forms the payload of the `200 OK` HTTP response.
In case the query parameter does not exist, the response is constructed to return the HTTP status `400 Bad Request` with an error message.

## Preparing the Local Environment

It is possible to run the Azure Function locally, if some requirements are met.
The most important part ist to install the Azure Function Core Tools as described in the [documentation][core-tools-installation].

For macOS the installation can be done using [Homebrew][homebrew] using the commands below.

```shell
brew tap azure/functions
brew install azure-functions-core-tools@3
```

For deploying the Azure Function to the cloud, we need to install the Azure CLI as described in the [documentation][azure-cli-installation].
Using Homebrew on macOS, this means to run the command below.

```shell
brew install azure-cli
```

## Running the Azure Function Locally

Once the Azure Function is implemented, it makes sense to run it locally before deploying it to the cloud.
Local execution can either be handled by Intellij Idea or by Maven using the Azure Function Maven plugin configured earlier.

In case of using Intellij Idea, you can run the Azure Function by clicking on the play symbol in the Azure Function handler class.

![Run the Azure Function from Intellij Idea](/img/functions-run-function-intellij.png){: .center-image }

This may produce an error in which Intellij complains about not finding the Azure Functions Core Tools.

> Failed to validate runtime of function(null),Azure Functions Core Tools not found. Please go to [https://aka.ms/azfunc-install](https://aka.ms/azfunc-install) to install Azure Functions Core Tools. If you have installed the core tools, please refer [https://github.com/microsoft/azure-tools-for-java/wiki/FAQ](https://github.com/microsoft/azure-tools-for-java/wiki/FAQ) to get the core tools path and set the value in function run configuration..

To fix this, we need to find out the location of the Azure Function Core Tools.
In case of macOS or Linux, this can be achieved with `which func`, whereas `where func` should do the job on Windows.
Then configure the identified path in the run configuration in Intellij Idea and try again.

![Run configuration for running the Azure Function from Intellij Idea](/img/functions-run-configuration-intellij.png){: .center-image }

If you would like to run the Azure Function locally without using the Intellij Idea integration, you can execute `mvn package azure-functions:run` in the project root directory.
This will package and run the Azure Function using the Azure Functions Maven plugin.

Disregarding how you launch the application, you should see output similar to the one shown below once the Azure Function is ready.

```shell
[INFO] Azure Functions Core Tools found.

Azure Functions Core Tools
Core Tools Version:       3.0.3442 Commit hash: 6bfab24b2743f8421475d996402c398d2fe4a9e0  (64-bit)
Function Runtime Version: 3.0.15417.0

[2021-08-30T14:48:36.800Z] Worker process started and initialized.

Functions:

	greetings: [GET] http://localhost:7071/api/greetings

For detailed output, run func with --verbose flag.
[2021-08-30T14:48:41.893Z] Host lock lease acquired by instance ID '000000000000000000000000DD4D073D'.
```

In case of the sample Azure Function, the runtime will expose the HTTP endpoint at [http://localhost:7071/api/greetings](http://localhost:7071/api/greetings).
Opening the URL in the browser, the error message informing us about the missing `name` query parameter should appear.
Providing this parameter like with [http://localhost:7071/api/greetings?name=mimacom%20Blog%20Visitor](http://localhost:7071/api/greetings?name=mimacom%20Blog%20Visitor) should return the greeting message.

## Debugging the Azure Function Locally

In case you use the Intellij Idea integration to launch the Azure Function, you can make use of the integrated feature to run the launch configuration with debugging enabled.

If you launch the Azure Function with Maven, please use `mvn package azure-functions:run -DenableDebug` to run it in debug mode.
If everything works fine, you should see additional console output like shown below.

```
[2021-08-30T15:58:21.821Z] Worker process started and initialized.
[2021-08-30T15:58:21.904Z] Listening for transport dt_socket at address: 5005
```

Now we can attach a remote debugger by creating the respective run configuration in Intellij Idea and running it.

![Run configuration for running the remote debugger from Intellij Idea](/img/functions-remote-debugger-intellij.png){: .center-image }

## Deploying the Azure Function

One possibility to deploy an Azure Function to the cloud is to make use of the Azure Functions Maven plugin, which is already configured for the project.

To deploy, please run `mvn azure-functions:deploy`.
Your browser will pop up and ask you to log in.
Once you are logged in, you can close the browser and switch back to your terminal.
You will recognize, that the deployment started to create the necessary Azure resources.
This includes:

* A new Resource Group, in which all other resources will be created.
* An App Service Plan, which provides the necessary computation and memory resources to run the Azure Function.
* A Function App, which will later on contain the Azure Function.
* A Storage Account, which contains the files defining the Azure Function including the custom implementation.
* An Application Insights instance, which provides the capability to monitor the Azure Function.

Once all resources are created, the deployment starts.
During this process, the locally built and packaged Azure Function zip archive is uploaded to the Storage Account.
The Function App will then load the content of the zip file to run the Azure Function once triggered.

Finally, some output related to the one below should appear.
It provides the URL of the deployed Azure Function.
To test the function, you can open this URL in your browser.

```shell
[INFO] Starting deployment...
[INFO] Trying to deploy artifact to baitando-azure-functions-sample...
[INFO] Successfully deployed the artifact to https://baitando-azure-functions-sample.azurewebsites.net
[INFO] Deployment done, you may access your resource through https://ms.portal.azure.com/#@/resource/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azure-samples/providers/Microsoft.Web/sites/baitando-azure-functions-sample
[INFO] Syncing triggers and fetching function information
[INFO] HTTP Trigger Urls:
[INFO] 	 greetings : https://baitando-azure-functions-sample.azurewebsites.net/api/greetings
```

## Conclusion

This blog post described the first basic steps in implementing a serverless function as Azure Function with Java.

My main intention for writing about this topic is, that serverless function as a service approaches conceptually look very promising to me.
Since the technology in this area is pretty young, I wanted to get a better understanding of a related offering by simply trying it out with the programming language I am comfortable with.

This for sure does not mean, that running an Azure Function implemented with Java is always a good idea when providing e.g. a REST API.

[homebrew]: https://brew.sh/
[core-tools-installation]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=macos%2Ccsharp%2Cportal%2Cbash%2Ckeda#v2
[azure-cli-installation]: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
[sample-plain]: https://github.com/baitando/azure-samples/tree/master/azure-functions
[microsoft-tutorial]: https://docs.microsoft.com/en-us/azure/developer/java/spring-framework/getting-started-with-spring-cloud-function-in-azure
[host-file]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json
[settings-file-local]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-local#local-settings-file
[settings-file-global]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-app-settings
[override-settings]: https://docs.microsoft.com/en-us/azure/azure-functions/functions-app-settings
