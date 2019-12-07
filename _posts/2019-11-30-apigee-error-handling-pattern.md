---
layout: post
title:  "A Pattern for Apigee Error Handling"
category: IT
tags: apigee api
bigimg: /img/bigimg_apigee-error-handling-pattern.jpg
credname: Med Badr Chemmaoui
credurl: https://unsplash.com/photos/ZSPBhokqDMc
comments: true
origin: https://blog.mimacom.com/apigee-error-handling-pattern/
excerpt: Consistent error handling is an important issue when it comes to REST API design. It can make sense to deliver additional error information. This blog post describes how providing such information can be enforced when Apigee is used as API gateway.
---

Consistent error handling is an important issue when it comes to REST API design.
If HTTP is used as transport protocol, it is obvious to stick to specified semantics like using proper HTTP status codes in responses.
Especially in error situations it can make sense to deliver additional information in the response payload.
This blog post will have a closer look on a pattern which describes how providing such information can be enforced when [Apigee][apigee-website] is used as API gateway.

The ideas described here are based on a [community forum post][apigee-community-pattern-1] which was enhanced in [another one][apigee-community-pattern-2].
The pattern presented here is what I would consider a practical solution to realize consistent error representation including some more, valuable details in the payload.
Everything described here makes use of standard Apigee mechanisms.

The blog post describes the topic in detail.
It starts with a general introduction related to architecture and API design with strong focus on error representation.
Afterwards, it describes how error handling can be realized on different maturity levels and uses a sample which is implemented in a [GitHub repository][github-sample].
Finally, the pattern is described in a nutshell using some code snippets.

# Apigee as Part of the System Architecture

This blog post bridges the gap between conceptional thoughts related to error handling in REST APIs and the concrete implementation using Apigee, which is a widely used product in the area of API gateways.

In fact an API gateway like Apigee is not really the implementation of an API, but rather a platform which is located between the client using tha API and the backend system providing the implementation of the API.
One of its purposes is to handle cross-cutting concerns like security, monetization and throttling by exposing proxies for the backend system.

Such proxies are good candidates to enforce consistent error handling and a common error data format without the necessity to change the actual backend system.
That's the reason why it is worth to spend some thoughts on how this can be achieved with Apigee.

# API Design

For better understanding, the blog post is based on a sample use case.
The REST API exposed on Apigee provides a single resource for news which is available on `/news`.
It supports to get a news entry by its ID sending a `GET` request to a path following the schema `/news/{{id}}`
Such a news entry consists of a title and the content.

The `Accept` header specified in the request defines, which format to use for the data returned.
In case of `application/json`, the data is provided as JSON like shown below.

```json
{
  "title": "A Pattern for Apigee Error Handling",
  "content": "Consistent error handling is an important issue when it comes to REST API design..."
}
```

If the client sets the `Accept` header to `application/xml`, the response data looks similar to the snippet shown below.

```xml
<news-entry>
    <title>A Pattern for Apigee Error Handling</title>
    <content>Consistent error handling is an important issue when it comes to REST API design...</content>
</news-entry>
```

The status code provided in the response must be compliant with the HTTP specification.
Pages like the [Mozilla documentation][mozilla-statuscodes] provide helpful overview tables.
Additionally, a proper, human readable but short reason phrase is returned in conjunction with the status code.
This applies to all responses in all situations and should be the absolute minimum when designing a REST API.

Talking about the happy path, the status code `200` should be returned together with the reason phrase `OK`.
In this case, the response payload contains the news entry data in the desired format.

Besides this success scenario, there are several error situations which can occur and should be considered.

* The requested news entry does not exist.
Therefore `404` is returned as status code.
The reason phrase is `News Entry Not Found`.
* The `Accept` header is not specified in the request.
Therefore `406` is returned as status code.
The reason phrase is `Missing Accept Header`.
* The `Accept` header is set to something else than `application/json` or `application/xml`.
Therefore `406` is returned as status code.
The reason phrase is `Unsupported Accept Header`.

Access to the news should only be possible if the user is authorized properly using HTTP basic authentication, i.e. the relevant data is contained in the `Authorization` header of the request.
In addition to the already listed ones, there are the following error cases:

* The `Authorization` header is not set.
Therefore `401` is returned as status code.
The reason phrase is `Unauthorized`.
* Checking the value of the `Authorization` turns out, that the user is not permitted to access the requested news entry.
Therefore `403` is returned as status code.
The reason phrase is `Forbidden`.

# Sample Repository

A sample repository with an API proxy implementing the pattern described here is available on [GitHub][github-sample].
To try it out, the following requirements need to be fulfilled.

* The sample can be deployed even on an Apigee trial account. There is a [sign up form][apigee-signup] available to get such a trial account within minutes.
* Node.js and npm need to be present on the machine used to deploy the sample to Apigee.
If this is not already available, it can be installed by following [this installation guide][npm-node-installation].

To deploy the sample it needs to be cloned to modify some settings related to the Apigee environment to use.

* Copy the `.env.example` file to `.env`.
The copied file must not be committed since it will contain credentials.
* Set proper values for the keys specified there.

To install the necessary dependencies for the deployment, the command below needs to be executed.

```bash
npm install
```

Once these preparation steps are finished, the deployment can be triggered with the command below.

```bash
npm run deploy
```

The deployment process is based on the [apigeetool npm package][apigee-apigeetool-npm].
Necessary deployment steps are implemented in the file `scripts/deploy.ts`.

Besides the API proxy code, the repository contains files which can be used to call the provided API using the [built-in HTTP client of Intellij Idea][intellij-restclient].

* The `calls/rest-client.rest` file defines the calls.
* The calls use some variables which are specific for the Apigee environment.
To specify proper values, the file `calls/rest-client.env.json.example` needs to be copied to `calls/rest-client.env.json`.
In the copied file it is necessary to fill in the desired values.

For the sake of simplicity, no backend system is connected.
The functionality described earlier is implemented as dummy proxy, which returns static data.
This is sufficient for the purpose of this blog post.

For the same reason, the authorization check is simulated.
The user is treated as authorized only if the username `dummy` and the password `letmein` are used for HTTP basic authentication.

# API Proxy Structure

A request handled by an API proxy on Apigee is processed in [several flows][apigee-flows].
The steps involved in handling a request are visualized in the diagram shown below.
It is based on a similar diagram shown in the [Apigee documentation][apigee-flows].

![Login screen](/img/apigee_flow-general.svg)

The `PreFlow` of the `ProxyEndpoint` is a good place for common logic like authorization and desired response format checks.

The request to retrieve a news entry by its ID can be implemented using a `ConditionalFlow` inside the `ProxyEndpoint` section.
It should be executed whenever a `GET` request for `/news/*` is received.
The sample makes use of this `ConditionalFlow` to prepare the static dummy data returned in the response.

# Apigee Error Handling

Apigee provides general [documentation][apigee-errorhandling] related to error handling.
Basically the [`RaiseFault` policy][apigee-policy-raisefault] is used to indicate that an error occurred.
This can happen either automatically by executed policies or manually, e.g. if a conditional step triggers a `RaiseFault` policy.

If an error occurs, the execution of the regular flow is stopped.
An error flow starts and checks, whether the API proxy defines a `FaultRule` which describes how to handle the particular error situation.
Such steps should follow the goal of providing a proper error response to the client. 

If no `FaultRule` is executed, Apigee considers the `DefaultFaultRule` which can be defined on the API proxy level.
It acts as a kind of general fallback logic, which gets executed if there was no specific `FaultRule` registered.
If the `alwaysEnforce` attribute of this element is set to `true`, the `DefaultFaultRule` gets executed always as last step - even if a `FaultRule` was already executed.

If neither a specific `FaultRule` nor the `DefaultFaultRule` was specified, Apigee takes care of the error.
It uses the data of the `RaiseFault` policy to derive a default error response.

Following the goal of specifying error responses as part of the API design, it makes sense to explicitly handle errors within the API proxy and not to rely on the Apigee behavior related to unhandled errors.

The next sections will give an overview of how the Apigee error handling mechanisms can be used to enforce consistent error data structures.
It starts with a small solution which ensures that proper HTTP status codes and reason phrases are used.
This will then be further improved.

# HTTP Specific Attributes in Error Response

According to the design of the REST API discussed here, proper HTTP status codes and reason phrases must be used as absolute minimum.
This section points out, how this can be achieved in case of Apigee handling the errors.

In case of automatically raised errors, everything should already be specified properly.
Documentation related to such errors and their status codes is available in the [Apigee Policy Error Reference][apigee-policyerrorref].

For manually raised errors, it is necessary to take care of the relevant settings in the corresponding `RaiseFault` policy XML file.
All possible settings are described in the [policy documentation][apigee-policy-raisefault].
The snippet below defines a custom `RaiseFault` policy with proper values for the status code and the reason phrase.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<RaiseFault async="false" continueOnError="false" enabled="true" name="RaiseFault.Forbidden">
    <DisplayName>Indicate that user is not permitted to access the resource</DisplayName>

    <FaultResponse>
        <Set>
            <StatusCode>403</StatusCode>
            <ReasonPhrase>Forbidden</ReasonPhrase>
        </Set>
    </FaultResponse>
</RaiseFault>
```

If the flow would now trigger the `RaiseFault.Forbidden` policy, Apigee would derive an error response with the status code and reason phrase set accordingly.

An example of a request and the response is shown below.
Please note that Apigee takes care of the response content type according to the request header.
In the sample case it was set to `application/json`.
If `application/xml` is specified, an equivalent XML representation will be returned.

```
GET http://<organization>-<environment>.apigee.net/apigee-errorhandling-sample/news/35711

HTTP/1.1 403 Forbidden
Date: Sat, 02 Nov 2019 21:33:05 GMT
Content-Type: application/json
Content-Length: 129
Connection: keep-alive

{
  "fault": {
    "faultstring": "Raising fault. Fault name : RaiseFault.Forbidden",
    "detail": {
      "errorcode": "steps.raisefault.RaiseFault"
    }
  }
}

Response code: 403 (Forbidden); Time: 2260ms; Content length: 129 bytes
```

This shows that it is possible to reach a minimum of consistent error responses by simply configuring manually triggered `RaiseFault` policies properly.

# More Details in Payload of Error Response

It could make sense to provide additional error information in the response payload using a custom data structure or one matching a pre-defined schema like [RFC-7807][rfc-7807].

For manually raised errors, the `RaiseFault` policy can set the response payload accordingly.
The sample below shows, how this could look like if a data structure compatible to RFC-7807 is used.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<RaiseFault async="false" continueOnError="false" enabled="true" name="RaiseFault.Forbidden">
    <DisplayName>Indicate that user is not permitted to access the resource</DisplayName>

    <FaultResponse>
        <AssignVariable>
            <Name>custom.error.detail</Name>
            <Value>You are not allowed to access this resource.</Value>
        </AssignVariable>
        <Set>
            <StatusCode>403</StatusCode>
            <ReasonPhrase>Forbidden</ReasonPhrase>
            <Payload contentType="application/problem+json" variablePrefix="@" variableSuffix="#">
                {
                    "type": "apigee-errorhandling",
                    "title": "@message.reason.phrase#",
                    "status": "@message.status.code#",
                    "detail": "@custom.error.detail#",
                    "instance": "@request.path#"
                }
            </Payload>
        </Set>
    </FaultResponse>
</RaiseFault>
```

A Sample request and the corresponding response for this case are shown below.
The automatically derived response message contains the error details in the payload like specified in the snippet above.

```
GET http://<organization>-<environment>.apigee.net/apigee-errorhandling-sample/news/35711

HTTP/1.1 403 Forbidden
Date: Sat, 02 Nov 2019 21:55:13 GMT
Content-Type: application/problem+json
Content-Length: 376
Connection: keep-alive

{
  "type": "apigee-errorhandling",
  "title": "Forbidden",
  "status": "403",
  "detail": "You are not allowed to access this resource.",
  "instance": "/apigee-errorhandling-sample/news/35711"
}
            

Response code: 403 (Forbidden); Time: 1887ms; Content length: 376 bytes
```

This goes further than the minimum described in the previous section and works fine for manually raised errors.
Obviously it is not possible that automatically raised errors lead to such a data structure if the error response is automatically derived by Apigee.
As a consequence, error responses would not be consistent at this stage and the format would be different for automatically and manually raised errors.

# Establish a Common Format for Error Details

The error response shown in the previous section looks pretty good so far.
Nevertheless there are some downsides.

* It is not possible to define multiple payload entries for different content types.
Using the snippet above will always return the payload as `application/problem+json` - even if the `Accept` header is set to `application/xml` or another value.
The `RaiseFault` policy is therefore specific for a response format.
If multiple ones need to be supported, the same number of copies of the `RaiseFault` would be necessary.
In general this tight coupling between error indication (triggering the `RaiseFault` policy) and error representation (deriving the response to send to the client) is a problem.
* This only works for custom `RaiseFault` policies.
In case of automatically raised errors, it is not possible to influence the representation in the response using the mechanisms discussed so far.
This contradicts the goal, because the automatically and manually raised errors would look different from a client point of view.

Both topics can be solved by decoupling error indication and error representation by making use of fault rules.
The first step is to stick to standard HTTP specific attributes and custom variables in the `RaiseFault` policy.
Looking at the sample, there was already a variable called `custom.error.detail` defined.
The payload data used in the previous section of the blog post needs to be removed.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<RaiseFault async="false" continueOnError="false" enabled="true" name="RaiseFault.Forbidden">
    <DisplayName>Indicate that user is not permitted to access the resource</DisplayName>

    <FaultResponse>
        <AssignVariable>
            <Name>custom.error.detail</Name>
            <Value>You receive a 'forbidden' error because you set the scenario query parameter to the respective value.</Value>
        </AssignVariable>
        <Set>
            <StatusCode>403</StatusCode>
            <ReasonPhrase>Forbidden</ReasonPhrase>
        </Set>
    </FaultResponse>
</RaiseFault>
```

The next step is to take care of the conversion to the desired error detail format.
Apigee is not able to do this automatically, because it has no clue about the one to use here.
Therefore some logic is needed, which gets triggered in case of an error and takes care of the conversion.
This is what the `DefaulFaultRule` element on the API proxy level is used for.

```xml
<DefaultFaultRule name="DefaultFaultRule">
    <Step>
        <Name>AssignMessage.ConvertErrorToPlaintext</Name>
        <Condition>(request.header.accept = null) or ((request.header.accept != "application/json") and (request.header.accept != "application/xml"))</Condition>
    </Step>
    <Step>
        <Name>AssignMessage.ConvertErrorToJson</Name>
        <Condition>(request.header.accept = "application/json")</Condition>
    </Step>
    <Step>
        <Name>AssignMessage.ConvertErrorToXml</Name>
        <Condition>(request.header.accept = "application/xml")</Condition>
    </Step>
    <AlwaysEnforce>true</AlwaysEnforce>
</DefaultFaultRule>
```

The snippet above shows how this could look like.
The most important thing is that `<AlwaysEnforce/>` is set to `true`.
This means, that the `DefaultFaultRule` is always triggered - even if another `FaultRule` was already executed.
Since the common error detail format should be enforced, this is necessary.

There are three conditional steps defined.

* One step creates a XML representation of the error if the `Accept` header is set to `application/xml`.
* One step creates a JSON representation of the error if the `Accept` header is set to `application/json`.
* One step creates a plaintext representation of the error if no or an unsupported `Accept` header is set.

The conversion is handled in separate `AssignMessage` policies.
As an example, the policy for JSON conversion is shown.
This is more or less similar to the parts which were removed from the `RaiseFault` policy earlier.
It picks up the variables and uses them to create the error detail data structure which gets assigned to the response payload.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage name="AssignMessage.ConvertErrorToJson">
    <DisplayName>Derive error details for JSON format</DisplayName>

    <Set>
        <Payload contentType="application/problem+json" variablePrefix="@" variableSuffix="#">
            {
                "type": "apigee-errorhandling",
                "title": "@message.reason.phrase#",
                "status": "@message.status.code#",
                "detail": "@custom.error.detail#",
                "instance": "@request.path#"
            }
        </Payload>
    </Set>

    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
</AssignMessage>
```

Decoupling error indication and error representation is solved.
The `RaiseFault` policy takes care of defining the HTTP specific attributes and additional variables, which are used to create the error representation in the fault rules of the API proxy.

The good thing is, that this will lead to a minimum acceptable representation of the error even if there is no fault rule for proper conversion registered.
If Apigee picks up the error to automatically derive the response, it will always contain a proper HTTP status code and reason phrase, because they are specified using the standard attributes in the `RaiseFault` policy. 

The same mechanism applies in case of automatically raised errors.
In this case, the `custom.error.detail` variable won't be set.
To cover this case, the `IgnoreUnresolvedVariables` should be set `true` in the `AssignMessage` policy which creates the error representation.
In case of automatically raised errors, the detail message is then empty in the response while all other values are set.
Therefore the goal of enforcing a common error detail data structure is achieved now.

# Optionally Enrich Automatically Raised Errors

As previously stated, the detail message is missing in case of automatically raised errors.
To get proper detail messages in such cases, it is possible to introduce additional fault rules on the API proxy level.
The `FaultRule` shown below is triggered if a fault with name `UnresolvedVariable` occured.
Please note that using the fault name in the condition is an approach which is recommended by Apigee in the [documentation][apigee-errorhandling].

```xml
<FaultRules>
    <FaultRule name="EnhanceInternalServerError">
        <Step>
            <Name>AssignMessage.InternalServerErrorDetail</Name>
        </Step>
        <Condition>(fault.name = "UnresolvedVariable")</Condition>
    </FaultRule>
</FaultRules>
```

If the condition is true, an `AssignMessage` policy is triggered.
Its purpose is to set all necessary custom variables referenced in the error detail data structure.
In case of the sample, it would have to set the `custom.error.detail` variable like shown below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage async="false" continueOnError="false" enabled="true" name="AssignMessage.InternalServerErrorDetail">
    <DisplayName>Add details for automatically raised internal server error</DisplayName>
    <AssignVariable>
        <Name>custom.error.detail</Name>
        <Value>Please check to find out why this error occurred.</Value>
    </AssignVariable>
</AssignMessage>
```

Using this, the error details message would be also available for this automatically raised error.
The sample below shows a matching request and its response.

```
GET http://<organization>-<environment>.apigee.net/apigee-errorhandling-sample/news/112

HTTP/1.1 500 Internal Server Error
Date: Sat, 02 Nov 2019 23:32:13 GMT
Content-Type: application/problem+json
Content-Length: 314
Connection: keep-alive

{
  "type": "apigee-errorhandling",
  "title": "Internal Server Error",
  "status": "500",
  "detail": "Please check to find out why this error occured.",
  "instance": "/apigee-errorhandling-sample/news/112"
}
        

Response code: 500 (Internal Server Error); Time: 1857ms; Content length: 314 bytes
```

# Reuse Error Conversion Logic

The mechanisms explained so far are limited to a single API proxy, which is not sufficient in many cases.
From an API consumer point of view, an API is the API product, which can consist of more than on API proxy.
To establish a consistent error representation across multiple proxies, the conversion logic can be put in a shared flow to make it reusable.
They can be developed and deployed separately.

In the sample, the shared flow for the error conversion could look like shown below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<SharedFlow name="default">
    <Step>
        <Name>AssignMessage.ConvertErrorToPlaintext</Name>
        <Condition>(request.header.accept = null) or ((request.header.accept != "application/json") and (request.header.accept != "application/xml"))</Condition>
    </Step>
    <Step>
        <Name>AssignMessage.ConvertErrorToJson</Name>
        <Condition>(request.header.accept = "application/json")</Condition>
    </Step>
    <Step>
        <Name>AssignMessage.ConvertErrorToXml</Name>
        <Condition>(request.header.accept = "application/xml")</Condition>
    </Step>
</SharedFlow>
```

To integrate the shared flow in a specific proxy, a `FlowCallout` policy is necessary.
It simply references the shared flow by its name which could look similar to the snippet shown one below.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<FlowCallout async="false" continueOnError="false" enabled="true" name="FlowCallout.ErrorConversion">
    <DisplayName>Delegate error conversion to the shared flow</DisplayName>
    <SharedFlowBundle>error-conversion</SharedFlowBundle>
</FlowCallout>
``` 

This policy is then triggered in the `DefaultFaultRule` in each proxy.
As a result, the shared flow will be triggered as last step whenever an error occured. 

```xml
<DefaultFaultRule name="DefaultFaultRule">
    <Step>
        <Name>FlowCallout.ErrorConversion</Name>
    </Step>
    <AlwaysEnforce>true</AlwaysEnforce>
</DefaultFaultRule>
```

The result from a client point of view is unchanged.
As a good feature in the area of debugging, the steps which are part of a shared flow, are also shown in the trace view.
Having a look at the screenshot below, the policies related to error conversion are show and grouped in a box which represents the shared flow.

![Proxy Execution Including Error Conversion in Shared Flow](/img/apigee_screenshot_shared-flow.png)

# Ideas for Further Optimization

What is right now available is a reuse solution, which requires that each proxy correctly implements the default fault rule to trigger the conversion.
Based on this, the final step to perfection would be a solution, in which this behavior could be automatically enforced without any implementation in the proxy.

An idea to solve this was to check, if fault rules and a default fault rule are also available in shared flows.
If this would exist, it would be possible to register the shared flow as a flow hook, which is automatically triggered e.g. before the regular processing starts or after it finished.

It turned out, that such a solution is not possible - at least not using fault rules.
A shared flow is simply a reusable sequence of conditional steps, which can be integrated in a proxy.
At execution time this looks similar to steps directly added in the proxy, i.e. the proxy is still around the shared flow and therefore responsible for error handling with fault rules.
This fits to the fact, that there are no fault rules available for shared flows.

Besides such a not realizable enforced runtime dependency between the proxy and a shared flow, there are some other alternatives.
The following list is inspired by a [community forum post][apigee-community-sharedflows].

* Include such logic during build time.
It would be possible to run a build process which adds a proper default fault rule to each proxy.
The necessary steps for the conversion could be added directly or a trigger for a shared flow could be added.
* Provide templates and design patterns.
It could make sense to provide such templates to offer a quickstart path for developing new proxies.
Such a template could contain the desired error handling implementation.
In addition it's often useful to document patterns for different use cases like the one of consistent error data details.
The blog post and the sample repository is an example.

# The Pattern in a Nutshell

Much explanation was given in this blog post.
To make this easier to use, a short overview is given below.

* Error handling is an important topic to consider during REST API design.
As a minimum proper HTTP status codes and reason phrases should be used.
To further improve this, it could make sense to provide error details in a consistent way.
* For manually triggered errors, the `RaiseFault` policies should define semantically correct HTTP status codes and understandable reason phrases using the standard attributes `StatusCode` and `ReasonPhrase`.
Additional, custom attributes for error details should also be added directly in the `RaiseFault` policy configuration.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<RaiseFault async="false" continueOnError="false" enabled="true" name="RaiseFault.Forbidden">
    <DisplayName>Indicate that user is not permitted to access the resource</DisplayName>

    <FaultResponse>
        <AssignVariable>
            <Name>custom.error.detail</Name>
            <Value>You receive a 'forbidden' error because you set the scenario query parameter to the respective value.</Value>
        </AssignVariable>
        <Set>
            <StatusCode>403</StatusCode>
            <ReasonPhrase>Forbidden</ReasonPhrase>
        </Set>
    </FaultResponse>
</RaiseFault>
```

* For each automatically triggered error, a `FaultRule` should be added in order to provide additional, custom attributes for error details in a dedicated `AssignMessage` policy.

```xml
<FaultRules>
    <FaultRule name="EnhanceInternalServerError">
        <Step>
            <Name>AssignMessage.InternalServerErrorDetail</Name>
        </Step>
        <Condition>(fault.name = "UnresolvedVariable")</Condition>
    </FaultRule>
</FaultRules>
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage async="false" continueOnError="false" enabled="true" name="AssignMessage.InternalServerErrorDetail">
    <DisplayName>Add details for automatically raised internal server error</DisplayName>
    <AssignVariable>
        <Name>custom.error.detail</Name>
        <Value>Please check to find out why this error occurred.</Value>
    </AssignVariable>
</AssignMessage>
```

* A `SharedFlow` with the purpose of error conversion should be provided.
It takes care of deriving a proper and consistent error representation.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<SharedFlow name="default">
    <Step>
        <Name>AssignMessage.ConvertErrorToPlaintext</Name>
        <Condition>(request.header.accept = null) or ((request.header.accept != "application/json") and (request.header.accept != "application/xml"))</Condition>
    </Step>
    <Step>
        <Name>AssignMessage.ConvertErrorToJson</Name>
        <Condition>(request.header.accept = "application/json")</Condition>
    </Step>
    <Step>
        <Name>AssignMessage.ConvertErrorToXml</Name>
        <Condition>(request.header.accept = "application/xml")</Condition>
    </Step>
</SharedFlow>
```

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<AssignMessage name="AssignMessage.ConvertErrorToJson">
    <DisplayName>Derive error details for JSON format</DisplayName>

    <Set>
        <Payload contentType="application/problem+json" variablePrefix="@" variableSuffix="#">
            {
                "type": "apigee-errorhandling",
                "title": "@message.reason.phrase#",
                "status": "@message.status.code#",
                "detail": "@custom.error.detail#",
                "instance": "@request.path#"
            }
        </Payload>
    </Set>

    <IgnoreUnresolvedVariables>true</IgnoreUnresolvedVariables>
</AssignMessage>
```

* Each API proxy should define a `DefaultFaultRule` with `AlwaysEnforce` set to `true`.
It should trigger the `SharedFlow` for error conversion.
* The easiest way to share this across teams is most likely to provide a template repository including an API proxy with proper and consistent error handling together with a pattern description.
If this is not enough, build time inclusion could be worth a look.

# Conclusion

This blog post had a closer look at error handling in Apigee API proxies.
The mechanisms provided by the platform were explained and used in a way, which makes it possible to reach consistent error handling and representation.

Please feel free to consider the thoughts described here as well as the sample code provided in the [GitHub repository][github-sample]. 

[npm-node-installation]: https://docs.npmjs.com/downloading-and-installing-node-js-and-npm
[apigee-website]: https://cloud.google.com/apigee/
[apigee-signup]: https://login.apigee.com/sign__up
[apigee-apigeetool-github]: https://github.com/apigee/apigeetool-node
[apigee-apigeetool-npm]: https://www.npmjs.com/package/apigeetool
[apigee-flows]: https://docs.apigee.com/api-platform/fundamentals/what-are-flows
[apigee-errorhandling]: https://docs.apigee.com/api-platform/fundamentals/fault-handling.html
[apigee-policy-raisefault]: https://docs.apigee.com/api-platform/reference/policies/raise-fault-policy
[apigee-policy-verifyapikey]: https://docs.apigee.com/api-platform/reference/policies/verify-api-key-policy
[apigee-policy-javascript]: https://docs.apigee.com/api-platform/reference/policies/javascript-policy
[apigee-policyerrorref]: https://docs.apigee.com/api-platform/reference/policies/error-code-reference
[github-sample]: https://github.com/baitando/apigee-samples/tree/error-handling-post
[exec-order-faultrules-inconsistentn]: https://community.apigee.com/questions/53592/execution-order-of-fault-rules.html
[flow-variables]: https://docs.apigee.com/api-platform/reference/variables-reference
[mozilla-statuscodes]: https://developer.mozilla.org/de/docs/Web/HTTP/Status
[rfc-7807]: https://tools.ietf.org/html/rfc7807
[intellij-restclient]: https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html
[apigee-community-pattern-1]: https://community.apigee.com/articles/23724/an-error-handling-pattern-for-apigee-proxies.html
[apigee-community-pattern-2]: https://community.apigee.com/articles/47312/an-improved-pattern-for-fault-handling.html
[apigee-community-sharedflows]: https://community.apigee.com/articles/34410/achieving-awesome-reuse-with-shared-flows-and-flow.html
