---
layout: ../../layouts/Post.astro
title: What Five-Nines Reliability Really Takes
description: How controlled load testing exposed bottlenecks across a .NET service, Elasticsearch, Redis, and Kubernetes.
date: '2026-09-20'
---

The goal seemed clear: scale a platform handling approximately 100 transactions per second to 1,000 TPS and beyond.

The hard part was nobody knew if the platform was able to do it.

That was 2022. Traffic came in with batch sizes from 1 to 200 items, and we were about to launch a new API that was going to require more processing power than the existing one. Availability was below 99.9%—the platform suffered from multiple outages in the six months since I joined the project. We had a production environment, Kubernetes, application logs and AWS API Gateway access logs. What we didn't have was a way to find out the most important thing:

**What is the weakest point of the platform?**

We thought the answer should have been in the application capacity. That was only half of the picture.

## The first limitation was the lack of a test

API Gateway access logging provided some insight into what traffic we had on the platform. With CloudWatch Logs Insights, it was possible to query request rates and latencies and understand production behavior of the system. But this was not enough to understand the safe capacity limit of the platform.

Observed traffic is not equal to tested capacity.

To discover it, we had to generate controlled traffic, vary batch sizes and run tests long enough for queues and dependent systems to surface. I built a load testing harness using Azure Load Testing and created a test script to generate JMeter test files for both APIs.

The traffic pattern was defined by AWS API Gateway access logs. The practical task was how to generate and sustain that traffic pattern long enough. I tuned JMeter thread and loop settings until the test was able to sustain 30 minutes of load. One burst may show the platform hitting a number. A sustained test will show if it can stay there.

## Adding a pod did not add the capacity we wanted

The first significant limitation appeared at approximately 500 TPS. As CPU usage increased, latency started increasing rapidly. In the end, the liveness probes failed and application pods restarted.

Our natural reaction was to follow the obvious path—add another node and another pod.

The expected result was not achieved.

That changed the direction of the investigation. If additional application capacity doesn't increase the throughput, the application tier must be waiting for something else.

Telemetry data from the load test showed that wait times in Elasticsearch were increasing and its CPU usage was approaching 100%. The search tier became the limiting factor. Application replicas could not make a saturated dependency process requests more quickly.

We doubled the number of cores of the Elasticsearch VM and ran the test again.

The result improved, but only up to 700 TPS.

## The bottleneck returned to the application

At 700 TPS, the platform was still below the 1,000 TPS requirement. The next issue was found in the application's ability to process larger batches under higher traffic.

It wasn't visible at the initial traffic level. It surfaced with the combination of increased request rate and batch size.

The application team was able to investigate and fix the processing issue. After the change, the platform achieved the 1,000 TPS requirement.

It was easy to consider it an endpoint and the work as done. The test passed the goal number. The sustained load test wasn't finished with us yet.

## Fifteen minutes later, Redis became the problem

After approximately 15 minutes of sustained load testing, Redis connection failures started appearing in Kibana. The application worker-thread setting was exhausted. It caused liveness failures and another restart cycle of the application pods.

It was a different limitation from the previous Elasticsearch saturation problem. The platform was able to reach the target rate, but it couldn't sustain the conditions yet.

The solution included three related changes: the development team changed the StackExchange.Redis library, enabled its async capabilities, and I found a worker-thread value that worked with the Kubernetes resource requests and limits. The worker-thread value was supplied as a variable during the Helm upgrade.

The number itself was not the lesson. A thread setting that looks acceptable in isolation may not be suitable for the CPU limits of a container, the Redis client library and the way the application works.

## The capacity number was only a starting point

Eventually, the platform reached the required 1,000 TPS rate. Much more importantly, the investigation revealed our understanding of capacity.

The limiting factor shifted as we changed the system: first it was the application CPU usage and liveness behavior, then Elasticsearch saturation, then application batch processing and finally Redis connectivity and exhaustion of the worker-thread setting.

The platform was not a single component with a single capacity number. It was a request path with multiple limiting factors.

That's why an additional pod wasn't the capacity plan. The capacity test needed realistic batch sizes, representative traffic, telemetry of the dependent systems and enough time to expose saturation, queue growth, thread exhaustion and recovery behavior.

Lessons that I learned were very straightforward:

1. Measure the system you have before designing the system you want to build.
2. Test dependencies as part of the platform; application replicas can't hide a saturated search or cache tier.
3. Separate the sustained load test from a successful burst.
4. Take into account resource limits, client libraries, asynchronous processing and health probes together.
5. Capacity belongs to the whole request path, not to a single Kubernetes deployment.

This was only one step in our reliability journey. Five-nines availability wasn't achieved with one scaling decision. It was the result of discovering the place where our assumptions didn't match production reality and changing the system and its measurement together.
