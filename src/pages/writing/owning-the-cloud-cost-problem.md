---
layout: ../../layouts/Post.astro
title: Owning the Cloud Cost Problem
description: How taking ownership of an unmanaged cost problem reduced monthly cloud spend from AUD 700,000 to AUD 550,000.
date: '2026-09-21'
---

Nobody wanted to own the cloud bill. The number was too large to ignore but responsibility sat across subscriptions and different teams. When someone asked who would take it on I put my hand up. We started around seven hundred thousand a month in Australian dollars. The goal was not just numbers on a page. We had to figure out where the spend was actually going and stop the same waste coming back later.

The first problem was visibility. There was no central view that made costs easy to check. Finance held the invoices and the platform teams knew bits of the infrastructure but neither side could answer the real questions about which parts were driving the biggest numbers or whether they were still needed. I worked with finance and the central team to get cost dashboards set up in CloudHealth for Azure and Cloudability for AWS. That gave us a way to rank the largest items across everything before looking closer.

The dashboard was not the saving by itself. It just gave us a place to start asking better questions. We went after the highest cost subscriptions first then drilled down from there to the service and the resource. Each one needed an owner and a decision on whether to keep it or change it or remove it. Finding the actual resource was usually easier than finding who owned it. A cost line might sit in a platform subscription but the reason could have come from an old audit or a project that finished months ago.

That process brought up waste that was hard to spot when every team only looked at its own stuff. Azure Migrate was one case. The on premises work had finished but the service was still running and turning it off saved about twenty five thousand a month. Storage diagnostic logging was another. It had been turned on for an audit and never switched off once the requirement ended. Sorting that out took a few rounds of talks with the teams involved to confirm the history and get approval to stop it. That saved another fifteen thousand or so each month. Other savings came from right sizing and reservations where it made sense and removing disks that were not used.

The bigger issue though was governance. Before this solutions could get built without anyone really checking the long term cost impact. I helped set up a process where new proposals went through an architecture review that included expected operating costs. Platform teams started looking at the dashboards on a regular basis instead of waiting for a budget problem to appear. It turned cost work from a one time clean up into something that happened as part of normal engineering.

Monthly spend dropped from around seven hundred thousand to five hundred fifty thousand. That is about one point eight million less each year. I think the review board part was what helped most in the end but it is still early to say how well it holds up once people get used to the new flow.

| Measure | Before | After |
| --- | ---: | ---: |
| Monthly cloud spend | Approximately AUD 700,000 | Approximately AUD 550,000 |
| Monthly reduction | — | Approximately AUD 150,000 |
| Annualised reduction | — | Approximately AUD 1.8 million |
| Cost visibility | Fragmented across teams and invoices | Central dashboards across Azure and AWS |
| Cost governance | No consistent review process | Architecture review and recurring platform reviews |
