---
layout: ../../layouts/Post.astro
title: What Five-Nines Reliability Really Takes
description: How controlled load testing exposed bottlenecks across a .NET service, Elasticsearch, Redis, and Kubernetes.
date: '2026-09-20'
---

# What Five-Nines Reliability Really Takes

The target looked straightforward: take a platform handling roughly 100 transactions per second and make it handle more than 1,000.

The difficult part was that nobody could tell us whether the platform was capable of doing that.

This was 2022. Requests arrived in batches ranging from 1 to 200 items, and we were preparing to introduce a new API that required substantially more compute than the existing one. Availability had not been 99.9%; in the six months before I joined, the platform had experienced multiple outages. We had a production platform, Kubernetes, application logs, and AWS API Gateway access logs. What we did not have was a reliable answer to the question that mattered most:

**Where would the platform fail first?**

We expected the answer to involve application capacity. That turned out to be only part of the story.

## The first test was the absence of a test

AWS API Gateway access logging gave us a useful view of the traffic we were already receiving. With CloudWatch Logs Insights, we could query request rates and latency. That helped us understand production behaviour, but it did not tell us the safe operating limit of the platform.

Observed traffic is not the same as tested capacity.

We needed to generate controlled traffic, vary the batch sizes, and hold the load long enough for queues and downstream dependencies to reveal themselves. I built a load-testing harness using Azure Load Testing and created a script that generated JMeter test files for both APIs.

The traffic model came from the access logs. The practical challenge was making the test sustain that model. I had to tune the JMeter thread and loop settings until the test could hold a stable load for 30 minutes. A short burst could show that the system reached a number. A sustained test could show whether it could live there.

## Adding a pod did not add the capacity we expected

The first important limit appeared at around 500 TPS. Latency began to rise sharply as CPU increased. Eventually, the liveness probes failed and the application pods restarted.

Our first response was predictable: add another node and another pod.

The throughput did not increase as expected.

That result changed the investigation. If more application capacity was not producing more throughput, the application tier was probably waiting on something else.

The load-test telemetry showed that Elasticsearch wait times were increasing while its CPU was reaching saturation. The search tier had become the limiting factor. More application replicas could not make a saturated dependency respond faster.

We doubled the core capacity of the Elasticsearch VM and ran the test again.

The result improved, but only to around 700 TPS.

## The bottleneck moved back into the application

At 700 TPS, the platform still fell short of the 1,000 TPS target. The next investigation found issues in the application’s processing of larger batches under increased volume.

This was not visible at the original traffic level. It emerged when the request rate and batch size increased together.

The application team investigated and fixed the processing issues. After that change, the platform reached the 1,000 TPS target.

It would have been easy to stop there and call the work complete. The test had reached its headline number. The sustained test was not finished with us yet.

## Fifteen minutes later, Redis became the problem

After roughly 15 minutes of sustained testing, Kibana began showing Redis connectivity failures. The application worker-thread setting was being exhausted. That led to liveness failures and another cycle of pod restarts.

This was a different failure from the Elasticsearch saturation we had just addressed. The platform could reach the target rate, but it could not yet sustain the conditions reliably.

The fix involved three connected changes: the development team changed the StackExchange.Redis library, enabled its asynchronous features, and I found a worker-thread value that worked with the Kubernetes resource requests and limits. The value was supplied as a variable during the Helm upgrade.

The number itself was not the lesson. A thread setting that looked reasonable in isolation was not necessarily appropriate for the container’s CPU constraints, the Redis client library, or the way the application performed its work.

## The capacity number was only the beginning

The platform eventually reached the required 1,000 TPS rate. More importantly, the investigation changed our understanding of capacity.

The limiting factor moved as we changed the system: first application CPU and liveness behaviour, then Elasticsearch saturation, then application batch processing, and finally Redis connectivity and worker-thread exhaustion.

The platform was not one component with one capacity number. It was a request path with several interacting limits.

That is why adding a pod was not a capacity plan. A meaningful capacity test needed realistic batch sizes, representative traffic, downstream telemetry, and enough duration to expose saturation, queue growth, thread exhaustion, and recovery behaviour.

The lessons I carried forward were simple:

1. Measure the system you have before designing the system you think you need.
2. Test dependencies as part of the platform; application replicas cannot hide a saturated search or cache tier.
3. Treat sustained load as a different test from a successful burst.
4. Consider resource limits, client libraries, asynchronous processing, and health probes together.
5. Capacity belongs to the whole request path, not to a single Kubernetes deployment.

This was one step in a longer reliability journey. Five-nines availability did not come from one scaling decision. It came from repeatedly finding the place where our assumptions stopped matching production behaviour, then changing both the system and the way we measured it.
