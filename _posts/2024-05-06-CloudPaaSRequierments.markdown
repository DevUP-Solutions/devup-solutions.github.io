---
layout: post
title: "Requirements in a Cloud PaaS World"
description: In any IT project requirements is hard, we have seen this and worked to make it easer  | This post gives an intro how we think about it can be used as a guide for you.
date: 2024-05-06 09:00:00 +0200
categories: Management
tags: [AIS, Integration, Security, PaaS, Requriements ]
author: "Mattias Lögdberg"
comments: true
---

Putting up the correct requirements when building services on PaaS components in the cloud sounds like a trivial task and is often not thought of more than "you're the expert, you know best, solve it" or "put it on the network." But there is much more to it, and we can see that in several breaches that have occurred during the year. These range from Microsoft's own Storage breach due to leaked keys to APIs completely open and requiring no authorization (I've seen it too often).

## So, what do we need to do?

* Start by defining basic settings required.
    * Like requiring HTTPS, TLS 1.3, and so on. (Yes, this is obvious but still often forgotten.)
* Set different requirements based on the confidentiality or criticality of the service.
    * Do we require a network?
    * Do we require "zero" downtime?
    * Is it okay to use static keys?
    * What preventive accident settings are important for us?
* Then define these settings per resource type.
* Make guidelines/templates so developers can easily achieve these requirements.
* Find a way to follow this up.
    * In code review to ensure everything new is compliant.
    * In operations to identify resources that need to be updated.
* Continuously monitor changes in the cloud services you are using and update requirements and guidelines accordingly.

### Define basic settings
Here we define the simple but still important settings we need, something like:

* Function App
    * HTTPS
    * TLS 1.3
    * Use Application Insights for logging
    * Turn off FTP deploy (since you are using DevOps pipelines or GitHub Actions that don't need it)

* Storage Account
    * HTTPS
    * TLS 1.2 (TLS 1.3 not available yet, but needs to be updated soon)
    * No anonymous blob containers

Continue per resource type and then define what is needed per requirement, document, and create templates.

### What's important based on confidentiality or criticality
Ask yourself what data is important based on confidentiality and what flows are important based on criticality for your business.

Based on this, create at least 2-3 different requirement setups:

* Normal level
    * Focus on basic settings that are "good enough" based on cost and security for most cases.
* Confidentiality level
    * What level of security do we need for these? Is it enough to ensure we are not using any static keys, or do we need full network lockdown, and is the cost for this infrastructure okay?
* Critical level
    * Design for uptime and stability. We want these flows to be up and running all the time, and in the worst case, what do we need? A ready region, or is it good enough with a deployment procedure that can deploy the flow in X time?

Important factors in this part are that when setting requirements like "full lockdown" or availability "99.9%," it will come with a cost. Is this cost worth it, or can we find an almost equally good solution for a lot less?

### Make guidelines/templates
This part is often underrated. My experience is that if we don't have this in place, there will be shortcuts and creative solutions created. This is often because we developers are lazy; we do what we know and like, and when under stress, we will not have the time to look for how to solve this. It's important to take the time to create guidelines and templates to guide and lead the team.

### Follow-up
This is also an underrated part. If we are not following this up, we will not find the shortcuts and mistakes that are created over time. To ensure that the whole team is working towards the same goal, we need to see and follow up on the work. This creates both insight into where you are and a target to work against, something to collaborate as a team to achieve, and importantly, celebrate wins.

### Continuously
We have never been developing in a world that is changing so rapidly as the Cloud and PaaS services are doing. And we have never been in a time where we so often get new, more secure mechanisms to use to protect us. So we need to stay open-minded, learn to be updated and informed.

We are spoiled with updated material, so make sure to update guidelines and templates regularly. Having outdated guidelines or templates creates a feeling of "this is not so important," and that should be avoided.

## Sounds smart, but how do I get this started?
Helium is our own service for helping our customers achieve the above. Interested in learning more? Get in contact, and we can show you how we can help you.