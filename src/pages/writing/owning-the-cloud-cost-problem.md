---
layout: ../../layouts/Post.astro
title: Owning the Cloud Cost Problem
description: How taking ownership of an unmanaged cost problem reduced monthly cloud spend from AUD 700,000 to AUD 550,000.
date: '2026-09-21'
---

Nobody wanted to own the cloud bill.

The number was too large to ignore, but the responsibility was spread across subscriptions, accounts, platforms and application teams. When the question was asked—who is going to take ownership of the costs?—I put my hand up.

The starting point was approximately AUD 700,000 per month. The target was not a spreadsheet exercise. We needed to understand where the money was going, decide what was actually required and create a process that stopped the same waste from returning.

## The first problem was visibility

There was no central reporting system that made the cost picture easy to review. Finance had the invoices, and the platform teams knew parts of the infrastructure, but neither view answered the questions we needed to ask:

- Which subscriptions and accounts were driving the highest costs?
- What services made up those costs?
- Who owned them?
- Were they still required?
- Had the original reason for running them expired?

I worked with Finance and the central FinOps team to set up cost dashboards in CloudHealth for Azure and Cloudability for AWS. That gave us a consistent way to rank the largest cost items across the estate before investigating them in detail.

The dashboard was not the saving by itself. It gave us a place to start asking better questions.

## Follow the largest costs to their owners

We reviewed the highest-cost subscriptions and accounts first, then drilled down from the account to the service and from the service to the resource. Each item needed an owner and an answer: keep it, change it, or remove it.

Finding the resource was usually easier than finding the ownership. A cost line might belong to a platform subscription, but the reason for it could have come from an old audit, a project that had already closed or a business requirement nobody remembered clearly. The work became a chain of conversations: who requested this, what risk was it addressing, is that risk still present and who can approve changing it?

That process surfaced waste that was difficult to see when every team looked only at its own resources.

Azure Migrate was one example. The on-premises migration had completed, but the service was still running. Decommissioning it saved approximately AUD 25,000 per month.

Storage diagnostic logging was another. It had been enabled during an audit and never turned off after the audit requirement ended. Establishing that history took repeated conversations across the teams involved. We had to identify where the requirement came from, get the business owners comfortable removing it and then secure the engineering capacity to complete the change. Disabling the unnecessary long-running logging saved another approximately AUD 15,000 per month.

Other savings came from the usual engineering work: right-sizing resources, purchasing reservations where the usage pattern justified them and removing unused disks.

The important distinction was that every saving came from understanding the requirement first. We were not cutting capacity blindly. We were removing resources whose purpose had ended, matching capacity to actual demand and choosing a better commercial model for stable usage.

## Turn cost review into an operating process

The deeper problem was governance. Before this work, solutions could be designed and deployed without a consistent review of their long-term cost.

I helped establish a process in which proposed solutions were reviewed by an architecture review board, including their expected operating cost and the choices that influenced it. Members of the platform team reviewed the cost dashboards regularly and worked through the highest-cost items rather than waiting for a budget escalation.

That changed cost management from a one-time clean-up into a recurring engineering responsibility. The dashboards showed where to focus. The review process made ownership explicit. The architecture board created a point where cost could be considered before a design became expensive to change.

## The result

Monthly cloud spend fell from approximately AUD 700,000 to AUD 550,000—a reduction of about AUD 150,000 per month, or AUD 1.8 million on an annualised basis.

| Measure | Before | After |
| --- | ---: | ---: |
| Monthly cloud spend | Approximately AUD 700,000 | Approximately AUD 550,000 |
| Monthly reduction | — | Approximately AUD 150,000 |
| Annualised reduction | — | Approximately AUD 1.8 million |
| Cost visibility | Fragmented across teams and invoices | Central dashboards across Azure and AWS |
| Cost governance | No consistent review process | Architecture review and recurring platform reviews |

The most valuable outcome was not a single decommissioning decision. It was making cost visible enough to discuss, owned enough to act on and routine enough to manage continuously.

Cloud cost is an engineering problem when architecture, capacity and operational habits determine the bill. Someone still has to take the first step and make the problem discussable. In this case, I volunteered to own that step, then built the reporting and governance needed for the wider team to keep improving it.
