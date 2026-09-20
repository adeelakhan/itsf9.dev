---
title: What Five-Nines Reliability Really Takes
description: How controlled load testing exposed bottlenecks across a .NET service, Elasticsearch, Redis, and Kubernetes.
date: '2026-09-20'
---

In 2022, we had a scaling problem that could not be solved by choosing a larger number in a configuration file.

The existing platform handled roughly 100 transactions per second at peak, with batch sizes ranging from 1 to 200. A new requirement called for more than 1,000 TPS with the same batch-size range. We were also introducing a new API that was more compute-intensive than the existing one.

The uncomfortable part was that we did not have a reliable answer to a basic question: how much could the platform actually deliver?

## Production traffic was not a capacity test

The application was written in .NET and exposed through AWS API Gateway. Access logging and CloudWatch Logs Insights gave us useful information about observed traffic, including request rates and latency. That told us what customers were sending, but not what the platform could safely sustain.

We needed a repeatable test that could generate controlled traffic, vary batch sizes, and run long enough to expose problems that would not appear in a short burst.

I built a load-testing harness using Azure Load Testing and created a script to generate JMeter test files for both APIs. I used traffic patterns from API Gateway access logs as a starting point, then tuned the JMeter thread and loop settings until the test could sustain a representative load for 30 minutes.

That tuning was part of the engineering work. A test that reaches a target rate for a few seconds is not the same as a test that holds that rate long enough for queues, downstream dependencies, and resource limits to show their behaviour.

## The first bottleneck was not the one we expected

At the existing scale, the platform reached about 500 TPS before latency began increasing sharply. CPU pressure grew, liveness probes started failing, and application pods restarted.

Our first instinct was to add another node and pod. That did not produce the expected throughput. The application tier was not the only limit.

The load tests showed that Elasticsearch wait times were increasing while its CPU was reaching saturation. Adding application capacity could not make the downstream search tier respond faster. We doubled the core capacity of the Elasticsearch VM and tested again.

That improved the result, but only to around 700 TPS. The remaining limitation was in the application’s handling of higher batch sizes under increased volume. The application team investigated and fixed those issues, allowing the platform to reach the 1,000 TPS target.

## Sustained load exposed a second dependency failure

The test still had more to teach us. After roughly 15 minutes of sustained testing, Kibana showed Redis connectivity failures. The application’s worker-thread setting was being exhausted. The value was supplied as a variable during the Helm upgrade, and it led to liveness failures and another cycle of application pod restarts.

The fix involved more than changing one number. The development team changed the StackExchange.Redis library, enabled its asynchronous features, and found a thread setting that worked with the Kubernetes resource requests and limits.

The important detail was the interaction between those settings. A thread value that looked reasonable in isolation was not necessarily appropriate for the container’s CPU and memory constraints or the way the Redis client was being used.

## What the test changed

The platform reached the required 1,000 TPS rate, but the more valuable result was the understanding gained along the way. The limiting factor moved between the application, Elasticsearch, Redis, and Kubernetes health behaviour as we changed the system.

The work also changed how we thought about capacity. Adding a pod was not a capacity plan. A meaningful capacity test needed realistic batch sizes, representative traffic, downstream telemetry, and enough duration to expose saturation and recovery behaviour.

The main lessons were:

1. Measure the system you have before designing the system you think you need.
2. Test dependencies as part of the platform; application replicas cannot hide a saturated search or cache tier.
3. Use sustained, representative load. Short tests conceal queue growth, thread exhaustion, and liveness failures.
4. Treat resource limits, client libraries, asynchronous processing, and health probes as one operational system.
5. Capacity is a property of the whole request path, not a single Kubernetes deployment.

This was one part of the longer reliability journey. Five-nines availability did not come from one scaling decision. It came from repeatedly finding the place where our assumptions stopped matching production behaviour, then changing the system and the way we measured it.
