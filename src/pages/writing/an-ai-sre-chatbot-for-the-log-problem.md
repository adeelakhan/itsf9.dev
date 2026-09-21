---
layout: ../../layouts/Post.astro
title: An AI SRE Chatbot for the Log Problem
description: How I connected an AI agent to Elasticsearch, Jira and Confluence to reduce incident triage effort and find signal in a noisy log stream.
date: '2026-03-22'
---

We had all of our application logs in Elasticsearch. That gave us a lot of data, but it did not make incident investigation fast.

During a critical incident, an engineer still had to search through the logs manually to work out which component could be contributing to the problem. It could take up to 30 minutes before we had a useful direction for the investigation. In an outage, that delay matters.

The second problem was quieter. The volume of logs made it difficult to notice patterns that had not yet become an incident. We were drowning in events and missing signals that could have pointed to a future outage. More logs were not going to solve that problem. We needed a better way to ask questions of the information we already had.

## Start with the investigation workflow

I researched the available AI agent options and found that Experian had its own AI gateway, along with a Python library called `arabica-client` for connecting to it.

I used Chainlit as the user interface and FastMCP to connect the agent to the systems an SRE would normally consult during an investigation:

- Elasticsearch for application logs
- Jira for incident and ticket history
- Confluence for operational knowledge and documentation

The goal was not to build a chatbot that could make changes to production. The goal was to give an engineer a faster path from a question to relevant evidence.

An engineer could ask what had changed, which component was showing errors, whether a similar problem had happened before or what the documented runbook said. The agent could combine current log evidence with historical tickets and the existing knowledge base.

## Choose enough model for the job

The AI gateway provided access to a range of frontier models. I tested the workflow against the use case rather than choosing the most expensive model by default.

GPT-based models, including the 5.4 mini model available through the gateway, were sufficient for the investigation tasks. Using the smaller model that met the quality bar also kept token usage and operating cost under control.

The model was only one part of the system. The quality of the answers depended on the context supplied by the tools, the way results were grounded and the checks applied before an answer was shown to an engineer.

## Treat evidence as part of the answer

An SRE assistant cannot be trusted because it sounds confident. It needs to show where its answer came from.

I added grounding and provenance checks to reduce hallucinations. The agent was expected to base its response on retrieved log entries, Jira history or Confluence content, and to make that source context visible. When the available evidence was weak or incomplete, the response needed to make that uncertainty clear instead of presenting a guess as a diagnosis.

This was especially important when the agent connected information across systems. A log message could indicate a symptom, a Jira ticket could explain a previous incident and a Confluence page could describe the intended operating procedure. Those sources were useful together, but they were not interchangeable evidence.

## Reduce the monitoring burden

The chatbot reduced monitoring effort by giving the team a quicker way to sift through the log stream and connect current symptoms with existing operational knowledge. It helped with the first part of an investigation: narrowing the search, identifying likely components and finding related history.

That distinction matters. The agent was an investigation aid, not an autonomous incident commander. Engineers remained responsible for validating the evidence, deciding what action to take and making any production change.

The practical lesson was that AI became useful when it was connected to the systems where operational context already lived. The model provided the reasoning layer, MCP provided controlled access to the tools and provenance checks kept the result tied to evidence.

The next challenge is measuring the capability more rigorously: time to first useful hypothesis, accuracy of component identification, repeated investigations that can be automated and the signals found before they become incidents. The chatbot gave us a better interface to the data. Those measures will tell us how much operational value it creates.
