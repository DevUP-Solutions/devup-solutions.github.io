---
layout: post
title: "Azure Functions version support for c# .net"
description: Azure Functions c# .net versions| This post explains the different versions, roadmap and their supported timeline.
date: 2022-09-19 08:00:00 +0200
categories: functions maintainability
tags: [Azure Functions, Integration, Maintainability ]
author: "Mattias Lögdberg"
comments: true
---

Wow time flies! Azure functions apps version 4 will now support .NET 7! Sure it's just in isolated process [isolated process](https://docs.microsoft.com/azure/azure-functions/dotnet-isolated-process-guide) for now but it will be GA soon. The rate in which we are seeing new .NET versions is fast and the question is how does this effect us who have been running function apps for some time now?

### Azure Functions Roadmap
With new .NET versions rolling out the old versions need to go out of support, it's just natural. Microsoft recently released a post on the updated [Roadmap for .Net Functions](https://techcommunity.microsoft.com/t5/apps-on-azure-blog/net-on-azure-functions-roadmap-update/ba-p/3619066?s=09).
We will try to describe what this will mean for us that have implemented Functions with .Net.

There are 4 different versions of Functions app runtime that we can use in Azure and here is the current overview of what .NET version is used on what Functions app version. [Source](https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide#supported-versions)

| Functions runtime version | In-process (.NET class library) | Out-of-process (.NET Isolated)|
|---|---|---|
| Functions 4.x | .NET 6.0 | .NET 6.0, .NET 7.0 (preview), .NET Framework 4.8 (preview) |
| Functions 3.x | .NET Core 3.1 | .NET 5.02 |
| Functions 2.x	| .NET Core 2.13 | n/a |
| Functions 1.x | .NET Framework 4.8 | n/a |

With .NET Core 3.1 support ending 2022-12-13 the list of supported versions will be like this: [Dotnet Core Lifecycle](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core#lifecycle)

| Version | Today(2022-09-15) | From 2022-12-13 |
|---|---|---|
| Functions 4.x | supported .NET version | supported .NET version |
| Functions 3.x | supported .NET version (.NET Core 3.1) | x |
| Functions 2.x	| x | x |
| Functions 1.x | supported .NET version | supported .NET version |

Microsoft also states [Azure Functions runtime versions overview](https://docs.microsoft.com/en-us/azure/azure-functions/functions-versions?tabs=azure-cli%2Cin-process%2Cv4&pivots=programming-language-csharp) that .NET Framework 4.8 will come to version 4 (in preview now).


### What does this mean to me?
If you are running any Function apps on v1-v3 today, you should start planning an update to version 4 and if you are using .NET Core that means upgrade to .NET 6 to run your workload on a supported platform. Check out this [migration guide](https://docs.microsoft.com/azure/azure-functions/functions-versions?tabs=azure-cli%2Cin-process%2Cv4&pivots=programming-language-csharp).

### Summary
When using PaaS services, we are spared of a lot of maintenance work. But the solutions we build with Functions are still tied to a .NET version and we need to maintain and update to new versions over time in order to run supported workloads. It's time to start planning those upgrades. For more detailed releases view the [public roadmap](https://github.com/Azure/azure-functions-dotnet-worker/projects/2).

We help our customers track the versions of every *Function App*. Showing what Functions are running a version that is either unsupported or going to be unsupported to make it easier to keep track. In **Helium** this check is called *CH70003*.