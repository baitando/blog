---
layout: post
title:  "Using the OpenAPI Generator for Spring Boot"
category: IT
tags: api tools
bigimg: /img/bigimg_openapi-generator.jpeg
credname: Cesar Carlevarino Aragon
credurl: https://unsplash.com/photos/NL_DF0Klepc
comments: true
origin: https://blog.mimacom.com/using-the-openapi-generator-for-spring-boot/
excerpt: In the area of REST APIs, formal specifications serve as the contract and define, what the consumer can expect from the API. To ensure consistency of the contract and the implementation, some parts of the source code can be generated from the specification document.
---

The idea of an API-first approach is to treat APIs as first-class citizens by building the software product around APIs.
Formal specifications serve as the contract and define, what the consumer can expect from the API.
To ensure consistency of the contract and the implementation, some parts of the source code can be generated from the specification document.

In this blog post, I would like to have a closer look at the basic project setup for generating code from the API specification.
We assume, that the API is specified using the [OpenAPI][openapi-homepage] format.
This allows us to make use of [Swagger Codegen][swagger-codegen] to generate parts of the code for the Spring Boot provider and consumer applications.

The use case for the example is a tiny piece of a banking API, which allows an employee to retrieve the accounts which he or she has access to.
The example project is available on [GitHub][github-repo].

# Specify the Contract

An API specification is basically a document, which describes the API and acts as the contract between the provider and the consumers of an API.
Talking about REST APIs, such a document considers things like:

* Paths and data structures of resources.
* Supported HTTP methods.
* Request and response headers.
* Possible response status codes.
* Cross-cutting concepts like error handling and authentication etc.

A basic solution is to provide the API specification as a textual explanation without a strict format.
While this kind of specification is flexible and easy to create, it leaves much room for interpretation.
This could lead to misunderstandings.

To solve this, standardized formats were invented, which define a syntax for describing the characteristics of an API.
As a side effect, a standardized syntax makes the API specification machine-readable.
This allows to automate API related processes like visualizing the specification, generating code and others.

Examples of API specification formats in the area of REST APIs are:

* [OpenAPI Specification][openapi-homepage] (formerly known as [Swagger Specification][swagger-openapi])
* [RAML][raml-homepage]
* [API Blueprint][api-blueprint-homepage]

The OpenAPI format is currently one of the most important API specification formats and widely adopted.
Swagger provides several tools, which support the OpenAPI format. The tool suite contains:

* [Swagger Editor][swagger-editor] for designing OpenAPI specifications.
* [Swagger UI][swagger-ui] for visualizing OpenAPI specifications in an interactive user interface.
* [Swagger Codegen][swagger-codegen] for generating client and server code from OpenAPI specifications.

In addition to Swagger Codegen, there is the [OpenAPI Generator][openapi-generator-website].
A statement about the differences is available in the [OpenAPI Generator FAQ section][swagger-openapi-gen-difference].
Basically, the OpenAPI Generator is a fork of Swagger Codegen and driven by the community while Swagger Codegen is driven by the company SmartBear.

Because of the relevance of OpenAPI and the surrounding tooling, the focus is on OpenAPI here.
The OpenAPI specification below is an example matching the simplified banking use case.

```yaml
openapi: 3.0.1
info:
  title: mimacom Banking API
  description: This is a sample API used to illustrate the tooling around an API-first approach.
  version: 1.0.0
  x-audience: external-public
  x-api-id: 78e624f8-f73e-4bbe-a918-bda261fd13ec
  contact:
    name: Andreas Hirsch
    url: https://www.mimacom.com
    email: andreas.hirsch@mimacom.com
servers:
  - url: 'http://localhost:8080'
    description: dev
paths:
  /accounts:
    get:
      summary: Retrieve all accounts.
      operationId: getAccounts
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/GetAccountsResponse'
components:
  schemas:
    Account:
      type: object
      required:
        - iban
      properties:
        iban:
          type: string
    GetAccountsResponse:
      type: object
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/Account'
```

The given specification can be rendered using the Swagger UI tool.
It takes the OpenAPI specification as input and renders the contained information dynamically.
The result is an interactive client, which can make use of the API.
A sample screenshot is shown below.

![API specification visualized by Swagger UI](/img/openapi-gen_swagger-ui.png)

If you would like to have a look at the sample specification loaded in the Swagger Editor including the Swagger UI, you can directly access it [here][swagger-editor-preloaded].

# Implement the API Provider

Based on the API specification, we will now implement the corresponding provider.
This can either be done manually or by generating the basic structure from the API specification.
We follow the generator approach and make use of the OpenAPI Generator which supports various languages and frameworks like Spring Boot as generation targets.
It is possible to generate a software development kit, i.e. a library, which can be published and referenced as a dependency, or to directly generate the code within the project.
We go for generating the code directly in the implementation project.

The OpenAPI Generator can be used as command line tool or as plugin for build tools like Maven and Gradle.
The `pom.xml` file below shows the integration of the OpenAPI Generator plugin in a Maven project.
To generate the server code, you need to add a `plugin` definition similar to the one below.

```xml

<plugin>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator-maven-plugin</artifactId>
    <version>4.3.1</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <inputSpec>${project.basedir}/../../spec/banking.yaml</inputSpec>
                <generatorName>spring</generatorName>
                <library>spring-boot</library>
                <modelNameSuffix>Dto</modelNameSuffix>
                <generateApiTests>false</generateApiTests>
                <generateModelTests>false</generateModelTests>
                <configOptions>
                    <basePackage>com.baitando.openapi.samples.gen.springbootserver</basePackage>
                    <modelPackage>com.baitando.openapi.samples.gen.springbootserver.model</modelPackage>
                    <apiPackage>com.baitando.openapi.samples.gen.springbootserver.api</apiPackage>
                    <configPackage>com.baitando.openapi.samples.gen.springbootserver.config</configPackage>
                    <dateLibrary>java8</dateLibrary>
                    <delegatePattern>true</delegatePattern>
                    <interfaceOnly>true</interfaceOnly>
                </configOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

This definition causes the generation of the server-side API during the Maven build.
All settings of the generator are explained in more detail on the [OpenAPI Generator website][openapi-generator-website].

We will make use of Spring Boot as generation target.
This leads to an API interface per specified resource, which is then implemented by custom code.
In the example shown below, the `AccountsApi` interface is generated by the OpenAPI Generator.
The implementation of the interface in the `AccountController` is self-written code.

```java
package com.baitando.openapi.samples.gen.springbootserver.controllers;


import com.baitando.openapi.samples.gen.springbootserver.api.AccountsApi;
import com.baitando.openapi.samples.gen.springbootserver.model.AccountDto;
import com.baitando.openapi.samples.gen.springbootserver.model.GetAccountsResponseDto;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;

/**
 * Implementation of REST API which allows to manage accounts.
 */
@RestController
public class AccountController implements AccountsApi {

    @Override
    public ResponseEntity<GetAccountsResponseDto> getAccounts() {
        return ResponseEntity.ok(
                new GetAccountsResponseDto().items(
                        Arrays.asList(
                                createAccount("DEXXXX"),
                                createAccount("ENYYYY")))
        );
    }

    private AccountDto createAccount(String iban) {
        AccountDto account = new AccountDto();
        account.setIban(iban);
        return account;
    }
}
```

The implementation of the controller class is shortened to simply return static dummy data.

# Implement the API Consumer

The API provider approach can be adopted for the API consumer to generate some client code.

The `plugin` snippet below needs to be added to the `pom.xml` file in order to generate the client-side code during the Maven build.
For the example, a REST template based implementation is generated, which can easily be used in Spring Boot applications to consume the API.

```xml

<plugin>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator-maven-plugin</artifactId>
    <version>4.3.1</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <inputSpec>${project.basedir}/../../spec/banking.yaml</inputSpec>
                <generatorName>java</generatorName>
                <modelNameSuffix>Dto</modelNameSuffix>
                <generateApiTests>false</generateApiTests>
                <generateModelTests>false</generateModelTests>
                <library>resttemplate</library>
                <configOptions>
                    <basePackage>com.baitando.openapi.samples.gen.springbootclient</basePackage>
                    <modelPackage>com.baitando.openapi.samples.gen.springbootclient.model</modelPackage>
                    <apiPackage>com.baitando.openapi.samples.gen.springbootclient.api</apiPackage>
                    <invokerPackage>com.baitando.openapi.samples.gen.springbootclient.client</invokerPackage>
                    <dateLibrary>java8</dateLibrary>
                </configOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Once the client code is generated, it can be used to retrieve data from the previously implemented provider.
The `DefaultApi` class contains generated code.
It defines a Spring Bean, which is directly qualified for being injected to the custom service implementation.

```java
package com.baitando.openapi.samples.gen.springbootclient.service.internal;

import com.baitando.openapi.samples.gen.springbootclient.service.AccountService;
import com.baitando.openapi.samples.gen.springbootclient.service.model.Account;
import com.baitando.openapi.samples.gen.springbootclient.api.DefaultApi;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class RestAccountService implements AccountService {

    private final DefaultApi apiClient;

    public RestAccountService(DefaultApi apiClient) {
        this.apiClient = apiClient;
    }

    @Override
    public List<Account> getAccounts() {
        return AccountConverter.convertFromDto(apiClient.getAccounts());
    }
}
```

To keep the snippet small, the conversion logic from the API data structure to the internal one was outsourced from the service class.

# Conclusion

This blog post described an example of a basic project setup for an API-centric software development approach.
We based the setup on an OpenAPI specification of the REST API and made use of the OpenAPI Generator to generate some portions of source code for the API provider and API consumer.
The code generation was directly included in the project build process, which loads the OpenAPI specification and derives the necessary source code for the given language and framework.

Now, what are potential benefits of such a process?
From my point of view there are the following:

* The API specification is the master and defines the contract.
  Everything starts with the specification, and the implementation is derived from the specification.
  This is the main difference to generating the specification from the implementation and shifts the mindset towards the API-first approach.
* When using Java or similar programming languages, you get compile-safety.
  You can try this out by modifying the OpenAPI specification and running the build process.
  If there was an incompatible change like renaming the resource, the compilation process will fail.
* Consuming REST APIs can be further simplified by providing generated SDKs for various languages and technologies to the consumers.

Nevertheless, it's also important to mention, that generating code from the specification comes with some downsides.
Thoughtworks for example put "spec-based codegen" to "hold" on their [Tech Radar][techradar] back in 2017 and mentioned the risk of unmaintainable and untestable code as reason.

From my point of view, there is no silver bullet - but there are pros and cons, and the decision about which approach to use may depend on the context.
If you come to the conclusion, that the generator approach suits your use case, you can find the complete source code of the example project, including the OpenAPI specification, the Spring Boot API provider, and the Spring Boot API consumer, on [GitHub][github-repo].

[techradar]: https://www.thoughtworks.com/radar/techniques/spec-based-codegen
[github-repo]: https://github.com/baitando/openapi-samples/tree/main/generator
[openapi-homepage]: https://www.openapis.org/
[swagger-homepage]: https://swagger.io/
[swagger-editor]: https://swagger.io/tools/swagger-editor/
[swagger-editor-preloaded]: https://editor.swagger.io/?url=https://raw.githubusercontent.com/baitando/openapi-samples/main/generator/spec/banking.yaml
[swagger-ui]: https://swagger.io/tools/swagger-ui/
[swagger-codegen]: https://swagger.io/tools/swagger-codegen/
[raml-homepage]: https://raml.org/
[api-blueprint-homepage]: https://apiblueprint.org/
[openapi-generator-website]: https://openapi-generator.tech/
[swagger-openapi-gen-difference]: https://openapi-generator.tech/docs/faq#what-is-the-difference-between-swagger-codegen-and-openapi-generator
[swagger-openapi]: https://swagger.io/docs/specification/about/
