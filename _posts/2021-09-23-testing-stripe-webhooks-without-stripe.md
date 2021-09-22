---
layout: post
title:  "Testing Stripe Webhooks Without Stripe"
category: IT
tags: stripe
bigimg: /img/bigimg_idea.jpeg
credname: Sascha Bosshard
credurl: https://unsplash.com/photos/qhhp1LwvPSI
comments: true
origin: https://blog.mimacom.com/testing-stripe-webhooks-without-stripe/
excerpt: Stripe uses webhooks to notify about events like successfully finished payments.  While integrating Stripe for the first time, I asked myself how such webhooks can be tested properly without relying on Stripe during automated test execution. In this blog post you will learn how it works.
---

Stripe uses webhooks to notify about events like successfully finished payments.
While integrating Stripe for the first time, I asked myself how such webhooks can be tested properly without relying on Stripe during automated test execution.
In this blog post you will learn how it works.

In addition to the explanation in this blog post, you will find a complete sample implemented as Spring Boot application with Kotlin at [GitHub][sample-github].

## Anatomy of Stripe Webhooks

Making use of Stripe usually means that Stripe takes at least care of processing payments.
Stripe collects the payment details by directly involving the user and is responsible for finishing the payment in the background.
This means, that the synchronous user interaction ends at some point and that the process continues asynchronously in the background.

To get notified about the outcome, Stripe provides a webhook mechanism in which you publicly expose an HTTP endpoint listening for post requests.
You register this endpoint in Stripe, select the event types you want to get notified about, and Stripe will call the webhook endpoint with a payload data structure defined by Stripe.

Of course, you do not want to accept events from anyone who sends such a request to your webhook endpoint.
This is where the custom HTTP request header `Stripe-Signature` sent by Stripe comes into play.

The value of this header is the signature of the payload, which you can verify e.g. by using the Stripe SDK to ensure that you are processing a valid event message sent by Stripe.
The Stripe [documentation][stripe-webhook-docs] explains this in more detail.

## Testing Webhooks With Stripe

Stripe offers some development tooling, which includes the Stripe CLI and the integrated mechanism of testing webhooks locally without exposing the endpoint publicly.
Additionally, you can trigger events for testing the webhook integration.
Both topics are explained in the [documentation][stripe-cli-docs].

These mechanisms empower us to run individual developer tests as well as automated tests which involve Stripe, e.g. as part of system tests.
Especially combining this with fixtures as described in the [documentation][stripe-fixture-docs] is valuable.

## Webhook Testing Independent of Stripe

Being able to test webhooks directly using Stripe is good and necessary, but usually I like to test such functionality without introducing a runtime dependency for test execution.
Of course, testing an HTTP endpoint is no rocket science and easily possible.
The major topic I had to evaluate carefully was testing the event signature verification.

Having a closer look at the [documentation][stripe-signature-docs] I recognized, that using the Stripe SDK is the easiest way to check the signature - but not the only one.
As alternative, it is possible to manually check the signature following a specific algorithm, which is described on the same page.
This approach basically calculates the valid signature based on the shared Stripe key and compares the result to the provided signature.
If both match, it was a valid request from Stripe.
If they do not match, it was an invalid or unauthorized request.

In fact, calculating the valid signature is exactly what helps us to test the webhook without Stripe.
We can configure our webhook application with any key and use the same key in the test to calculate the valid signature, which can be used for the test.

After some more research to verify this idea, I even found a test in the [Stripe code base][stripe-test-webhook], which in fact does the same.
With Kotlin I ended using the method shown below, which gets the event payload and the key as input and returns the proper signature.

```kotlin
private fun generateSigHeader(payload: String, key: String): String {
    // Inspired by https://github.com/stripe/stripe-java/blob/master/src/test/java/com/stripe/net/WebhookTest.java
    val timestamp = getTimeNow()
    val payloadToSign = String.format("%d.%s", timestamp, payload)
    val signature = computeHmacSha256(key, payloadToSign)

    return String.format("t=%d,%s=%s", timestamp, Webhook.Signature.EXPECTED_SCHEME, signature)
}
```

## Conclusion

With that knowledge I was able to implement a test for the webhook HTTP endpoint, which covers the different cases which may occur.
This includes a `401 Unauthorized` response in case no signature was provided, `403 Forbidden` in case of an invalid signature and now also the `200 OK` response in case a valid signature was provided.

A sample project demonstrating this with a Spring Boot application implemented in Kotlin is available on [GitHub][sample-github].

[stripe-webhook-docs]: https://stripe.com/docs/webhooks
[stripe-cli-docs]: https://stripe.com/docs/stripe-cli/webhooks
[stripe-fixture-docs]: https://stripe.com/docs/cli/fixtures
[stripe-signature-docs]: https://stripe.com/docs/webhooks/signatures
[stripe-test-webhook]: https://github.com/stripe/stripe-java/blob/master/src/test/java/com/stripe/net/WebhookTest.java
[sample-github]: https://github.com/baitando/stripe-samples/tree/master/webhook-testing
