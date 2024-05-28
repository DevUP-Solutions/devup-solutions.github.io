---
layout: post
title: "What is Cyber Hygiene in AIS"
description: What is cyber Hygiene in Azure Integration Services. | This post gives an introductio to the concept Cyber Hygiene in Azure Integration Services.
date: 2024-05-30 09:00:00 +0200
categories: Cyber Hygiene, Security
tags: [AIS,Integration, Security ]
author: "Mattias Lögdberg"
comments: true
---

In today's digital landscape, cyber hygiene is more critical than ever. With the increasing complexity of cyber threats and the expanding attack surfaces due to remote work and cloud computing, organizations must prioritize cybersecurity. This post will delve into the fundamentals of cyber hygiene, its importance, and why starting with the basics is essential for robust cybersecurity.

### Why bother?
Let's start with this important part!

Braches are real, we are suffering from the threat daily. We know attackers starts with less effort solutions, meaning if you have easy weaknesses that will be the starting point.

Building services in Azure Integration Services span all from critical and sensitive data to public available data, this means we are managing a complex setup. Whenever there is a complex solution with many services there is a increased need of structural work. What would a small misstake open for you?

For **Optus** it was a devestating experince where 10 million customers personal data was stolen. [Read more on Optus Breach](https://www.bbc.com/news/world-australia-63056838)
This all to an API that they where unaware of was unprotected, when was the last time you checked your api's?

### Start with the Basics
There are alot of sofisticated ways but what **Optus** is a proof of, we need to have the basics in place. Basics are:

* Authentication in any form
* Allways Encrypted access 
* Highest possible encryption methods

What this translates to in Azure Integration Services is:

* Static keys, JWT tokens, certificates, basic auth and so on.
* Transportation encryption via HTTPS/SFTP and so on.
* Choosing the latest encryption TLS 1.3

### Where do I start?

Start by mapping up your services and see where you are, what services are using auhentication, what is enforcing HTTPS and highest posible TLS version. When you have that list, start updating the infrastructure as code templates you are using (bicep/arm/terraform) with the easy and small updates that is needed per service.

1. List all services - for each service:
    1. Work thru all services
    1. Check authentication
    1. Check enforcing of Encrypted transport (HTTPS/SFTP and so on)
    1. Check lowest TLS/SSL requriement and push towards TLS 1.3
    1. Update resource accordingly

Work thur em all and make sure you are compliant.

### One time enough?
No, this is work that need's to be done continiously, here is some tips to get this working continiously.
* Make sure to review all new services published
* Make sure Infrastructure as code templates are updated
* Mark the calendar for next time to redo the work.

### If I need some assistance?
We at DevUP are experts in this area and we use our service **Helium** to quickly scan the whole environment and get the instructions needed to update each resource based on resource type. 

Reach out and we can show how you get this information in minutes and continiously.
