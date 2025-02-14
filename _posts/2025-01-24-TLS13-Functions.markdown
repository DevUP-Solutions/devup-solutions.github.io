---
layout: post
title: "Azure Functions Timer trigger and TLS 1.3 Settings"
description: TLS 1.3 is currently rolling out in Azure and Azure Functions, there are some issues with TLS 1.3 and timer trigger.
date: 2025-01-24 09:00:00 +0200
categories: Security
tags: [TLS, Security]
author: "Mattias Lögdberg"
comments: true
---

# TLS 1.3 and Timer trigger issues

Upgrading to TLS 1.3 is a great move to increase security in your Function Apps, but there is some issues that comes with the timer trigger.

So by enabling the TLS 1.3 timer trigger becomes unpredictable.
We have several test functions running on Consumption app tier and all run the same code with a timer trigger evefy 5 minutes, but as the graph bellow shows you can clearly see that it's only 2 functions running consequently, rest is (configured with TLS 1.3) only running around midnight.

![Timer trigger events](/assets/images/2025/02/tlsgraph.png)

Here is the code:

```c#
using System;
using System.Collections;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;

namespace BicepRunTest
{
    public class RunMe
    {
        private readonly ILogger _logger;

        public RunMe(ILoggerFactory loggerFactory)
        {
            _logger = loggerFactory.CreateLogger<RunMe>();
        }

        [Function("RunMe")]
        public void Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer)
        {

            var functionAppName = Environment.GetEnvironmentVariable("appName");
            _logger.LogInformation($"{functionAppName} am running now!");

            if (myTimer.ScheduleStatus is not null)
            {
                _logger.LogInformation($"Next timer schedule at: {myTimer.ScheduleStatus.Next}");
            }
        }
    }
}

```

Going back to TLS 1.2 makes the timer trigger become predictable again.


**In Summary:** Upgrading to TLS 1.3 is essential for maintaining a secure, efficient, and reliable digital presence. However, due to the current instability in the timer trigger we think you should wait for Functions using timer trigger.

### How do I know if I have an issue?

If you have unpredictable timer executions in your Functions you should test moving back to TLS 1.2 and create a support ticket to raise awareness at Microsoft.


### Troubelshoot your trace
You can use application insights to trace your triggers and see if it matches the chronos expression.


### If I need some assistance?
We at DevUP are experts in this area, and we use our service **Helium** to quickly scan the whole environment and get the TLS versions per resource, we know and will let you know when new versions of TLS is released.

Reach out, and we can show you how to get this information in minutes and continuously.