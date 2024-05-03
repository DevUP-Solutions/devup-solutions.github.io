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

Putting up the correct requirmenets when building services on PaaS components in the cloud sounds like a trivial task and often not thought of more than "you know solve it" or "put it in network". But there is much more too it and we can see that on several breaches that has occured during the year. All from Microsft's own Storage breach due to leaked keys, to API's completly open and no authorization required (I've seen it too often).


## So what do we need to do?

* Start with defining basic settings required
    * Like require Https, TLS 1.3 and son. (yes this is obivous but still often forgotten)
* Set different requriements based on confidentiality or criticality of the service.
    * Do we require Network?
    * Do we require "zero" downtime?
    * Is it okey to use static keys?
    * What prevent accident settings is important for us?
* Then define these settings per resource type
* Make guidelines/templates so developers easily can achieve these requirments.
* Find a way to follow this up
    * In Code review to make sure everything new is compliant
    * In Operations to identify reasources that need's to be updated.
* Continiously follow changes on cloud services you are usng and update requirmeents and guidelines.


### Define basic settings
Here we define the simple but still important settings we need.
Something like:

* Function App
    * HTTPS
    * TLS 1.3
    * Use Application Insights for logging
    * Turn of FTP deploy (since you are using DevOPS pipeliense or Github actions that don't need it)

* Storage Account
    * HTTPS
    * TLS 1.2 (TLS 1.3 not available yet, but need to be updated soon)
    * No anonomus blob containers

Contiue per resource type and then define what is needed per requirmenet, document and create templates.

### What's important based on confidentiality or criticality
Ask yourself what data is important based on confidentiality and what flows are important based on criticality for your business.

Based on this create atleast 2-3 different requirment setups 

* Normal level
    * Focus on basic settings that is "good enough" based on cost and security for most cases.
* Confideniality level
    * What level of security do we need for these, is it enought to make sure we are not using any static keys or de we need full network lockdown and is the cost for this infrastructure okey.
* Critical level
    * Design for uptime and stability, we want these flows to be up and rnning all the time and in worst case what do we need? A ready region or is it good enoguth with a deployment procedure that can deploy the flow in X time?

Important factors in this part is that when setting requriments like "full lockdown" or availability "99.9%" it will come with a cost, is this cost worth it or can we do an almost as good solution for alot less?

### Make guidelines/templates 
This part is often underrated, my experience is that if we don't have this in place there will be shortcuts and creative solutions created. This is often since we developers are lazy we do waht we know and like and when under stress we will not have the time to look for how to solve this. 
It's important to take the time to create guidelines and templates to guide and lead the team.

### Follow up
This is also a underrated part, since if we are not following this up we will not find the shortcuts and misstakes that is create over time. To make sure that the whole team are workign towards the same goal we need to see and follow up the work. This creates both insight into where you are and a target to work against, something to collaborate as a team to achieve, and importantly cellebrate wins.

### Continiously
We have never been developing in a world that is changing so rapidly as the Cloud and PaaS services are doing. 
And we have never been in a time where we so often get new more secure mechanisms to use to protect us.
So we need to stay openminded learn to be updated and informed.

We are spolied with updated material so make sure to updated guidelines and templates regulary. Having outdated guidelines or templates creates a feeling of "this is not so important" and that should be avoided.



## Sounds smart but how do I get this started?
Helium is our own service for helping our customers to achiving the above, interested in learning more? 
Get in contact and we can show you how we can help you.