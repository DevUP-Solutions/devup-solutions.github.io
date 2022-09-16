---
layout: post
title: "Azure Functions version support for c# .net"
description: Azure Functions c# .net versions| This post explains the different versions and their supported timeline.
date: 2022-09-16 13:00:00 +0200
categories: functions maintainability
tags: [Azure Functions, Integration, Maintainability ]
author: "Mattias Lögdberg"
comments: true
---

Wow time flies! We are now seeing news on Azure Functions version 4 that they support .net 7 in c#. Sure it's just in (isolated process)[https://docs.microsoft.com/azure/azure-functions/dotnet-isolated-process-guide] for now but it will come soon. The rate we are seeing new .net versions is really fast and the real question is how does this effect us who has been running functions for some time now?

### Azure Functions Roadmap
With new versions old versions need to go out of support, it's just natural.
Microsoft recently released a post on the updated (Roadmap for .Net Functions)[https://techcommunity.microsoft.com/t5/apps-on-azure-blog/net-on-azure-functions-roadmap-update/ba-p/3619066?s=09].
Trying to describe what this whill mean for us that have implemented Functions with .Net.

There are 4 different versions of Functions runtime that we can use in Azure and here is the current overview of what .NET version is used on what Functions version: (source)[https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide#supported-versions]

| Functions runtime version | In-process (.NET class library) | Out-of-process (.NET Isolated)|
|---|---|---|
| Functions 4.x | .NET 6.0 | .NET 6.0, .NET 7.0 (preview), .NET Framework 4.8 (preview) |
| Functions 3.x | .NET Core 3.1 | .NET 5.02 |
| Functions 2.x	| .NET Core 2.13 | n/a |
| Functions 1.x | .NET Framework 4.8 | n/a |

We need to understand this since it gives us insight into our situation.

With .NET Core 3.1 going out of support 2022-12-13 we will have the list of supported versions like this: (Dotnet Core Lifecycle)[https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core#lifecycle]

| Version | Today(2022-09-15) | 2022-12-13 |
|---|---|---|
| Functions 4.x | supported | supported |
| Functions 3.x | supported (.NET Core 3.1) | x |
| Functions 2.x	| x | x |
| Functions 1.x | supported | supported |

We can also read from the (Azure Functions runtime versions overview)[https://docs.microsoft.com/en-us/azure/azure-functions/functions-versions?tabs=azure-cli%2Cin-process%2Cv4&pivots=programming-language-csharp] that .NET Framework 4.8 will come to version 4 (in preview now).


### What does this mean to me?
If you are running any Functions on v1-v3 today, you should start planning an update to version 4 and if you are using .NET Core that means upgrade to .NET 6 to run your workload on a supported environment.
Check out this (migration guide)[https://docs.microsoft.com/azure/azure-functions/functions-versions?tabs=azure-cli%2Cin-process%2Cv4&pivots=programming-language-csharp].

### Summary
When using PaaS services we are spared of alot of maintainance work. But the solutions we build with Functions are still tied to a .net version and that we need to maintain and update to new versions over time in order to run supported workloads. It's time to start planning those upgrades.


We help our customers track the versions of each and every *Function App*. Showing what Functions are running an verion that is either unsupported or going to be unsupported to make it easier to keep track. In **Helium** this check is called *CH70003*.