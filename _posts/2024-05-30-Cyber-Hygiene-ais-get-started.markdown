---
layout: post
title: "Start with the basic Cyber Hygiene in AIS"
description: What is cyber hygiene in Azure Integration Services. | This post gives an introduction to the concept Cyber Hygiene in Azure Integration Services.
date: 2024-05-28 09:00:00 +0200
categories: Cyber Hygiene, Security
tags: [AIS, Integration, Security]
author: "Mattias Lögdberg"
comments: true
---

In today's digital landscape, cyber hygiene is more critical than ever. With the increasing complexity of cyber threats and the expanding attack surfaces due to remote work and cloud computing, organizations must prioritize cybersecurity. This post will delve into the fundamentals of cyber hygiene, its importance, and why starting with the basics is essential for robust cybersecurity.

### Why bother?
Let's start with this important part!

Breaches are real, and we are suffering from the threat daily. We know attackers start with less effort solutions, meaning if you have easy weaknesses, that will be the starting point.

Building services in Azure Integration Services span all from critical and sensitive data to publicly available data. This means we are managing a complex setup. What would a small mistake open for you?

For **Optus**, it was a devastating experience where 10 million customers' personal data was stolen. [Read more on Optus Breach](https://www.bbc.com/news/world-australia-63056838)
This all came from an API that they were unaware of being unprotected. When was the last time you checked your APIs?

Do you know how your Azure Integration Services are configured in detail?

### Start with the Basics
There are a lot of sophisticated ways, but what **Optus** is proof of, we need to have the basics in place. Basics are:

* Authentication in any form
* Always Encrypted access
* Highest possible encryption methods

What this translates to in Azure Integration Services is:

* Static keys, JWT tokens, certificates, basic auth, and so on.
* Transportation encryption via HTTPS/SFTP, and so on.
* Choosing the latest encryption TLS 1.3

### Where do I start?

Start by mapping out your services and see where you are, what services are using authentication, what is enforcing HTTPS and the highest possible TLS version. When you have that list, start updating the infrastructure as code templates you are using (Bicep/ARM/Terraform) with the easy and small updates that are needed per service.

1. List all services - for each service:
    1. Work through all services
    1. Check authentication
    1. Check enforcing of Encrypted transport (HTTPS/SFTP, and so on)
    1. Check lowest TLS/SSL requirement and push towards TLS 1.3
    1. Update resources accordingly

Work through them all and make sure you are compliant.

### One time enough?
No, this is work that needs to be done continuously. Here are some tips to get this working continuously:
* Make sure to review all new services published
* Ensure Infrastructure as code templates are updated
* Mark the calendar for the next time to redo the work.

### If I need some assistance?
We at DevUP are experts in this area, and we use our service **Helium** to quickly scan the whole environment and get the instructions needed to update each resource based on resource type.

Reach out, and we can show you how to get this information in minutes and continuously.
