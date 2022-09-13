---
layout: post
title: "Azure Functions version support for c# .net"
description: Azure Functions c# .net versions| This post explains the different versions and their supported timeline.
date: 2022-09-15 08:00:00 +0200
categories: functions maintainability
tags: [Azure Functions, Integration, Maintainability ]
author: "Mattias Lögdberg"
comments: true
---

Wow time flies! We are now seeing news on Azure Functions version 4 that they support .net 7 in c#. Sure it's just in (isolated process)[https://docs.microsoft.com/azure/azure-functions/dotnet-isolated-process-guide] for now but it will come soon. The rate we are seeing new .net versions is really fast and the real question is how does this effect us who has been running functions for some time now?

### How's the support ahead for C# Functions
With new versions old versions need to go out of support, it's just natural.
This gives us an idea of what we need to do in the short term to run supported functions.
First we need to see what version support what frameworks: (source)[https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide#supported-versions]

| Functions runtime version | In-process (.NET class library) | Out-of-process (.NET Isolated)|
|---|---|---|
| Functions 4.x | .NET 6.0 | .NET 6.0, .NET 7.0 (preview), .NET Framework 4.8 (preview) |
| Functions 3.x | .NET Core 3.1 | .NET 5.02 |
| Functions 2.x	| .NET Core 2.13 | n/a |
| Functions 1.x | .NET Framework 4.8 | n/a |

With .NET Core and .NET 5 going out of support 2022-12-03 we will have the list of supported versions like this:

| Version | Today(2022-09-15) | After 2022-12-03 |
|---|---|---|
| Functions 4.x | supported | supported |
| Functions 3.x | supported | x |
| Functions 2.x	| supported | x |
| Functions 1.x | supported | supported |

We can also read from the (Azure Functions runtime versions overview)[https://docs.microsoft.com/en-us/azure/azure-functions/functions-versions?tabs=azure-cli%2Cin-process%2Cv4&pivots=programming-language-csharp] that .NET Framework 4.8 will come to version 4 (in preview now).

**Note** above is only considering **C#** functions.

### Why update my Functions?
They will probobly work for undefined time but it's allways good to be on the safe side an migrate in a controlled fashion rather than in a burning critical situation.
Take the time, plan for a migration and do the migration to .net 6 it's fairly easy from .net Core.

### Summary
When using PaaS services we are spared of alot of maintainance work. But the solutions we build with Functions are still tied to a .net version and that we need to maintain and update to new versions over time in order to run supported workloads.


We help our customers track the versions of each and every *Function App*. Showing what Functions are running an verion that is either unsupported or going to be unsupported to make it easier to keep track. In **Helium** this check is called *CH70003*.