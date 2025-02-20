---
layout: post
title: "Azure Functions Timer Trigger and TLS 1.3 Settings"
description: TLS 1.3 is currently rolling out in Azure and Azure Functions. There are some issues with TLS 1.3 and the timer trigger.
date: 2025-01-24 09:00:00 +0200
categories: Security
tags: [TLS, Security]
author: "Mattias Lögdberg"
comments: true
---

# TLS 1.3 and Timer Trigger Issues

Upgrading to TLS 1.3 is a great move to increase security in your Function Apps, but there are some issues that come with the timer trigger.

By enabling TLS 1.3, the timer trigger becomes unpredictable.  
We have several test functions running on the Consumption app tier, all executing the same code with a timer trigger every 5 minutes. However, as the graph below shows, you can clearly see that only two functions run consistently, while the rest (configured with TLS 1.3) only execute around midnight.

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

Reverting to TLS 1.2 makes the timer trigger predictable again.

## In Summary
Upgrading to TLS 1.3 is essential for maintaining a secure, efficient, and reliable digital presence. However, due to the current instability of the timer trigger, we recommend waiting before enabling TLS 1.3 for Functions using the timer trigger.

### How Do I Know If I Have an Issue?
If you experience unpredictable timer executions in your Functions, try reverting to TLS 1.2 and create a support ticket to raise awareness at Microsoft.

### Troubleshoot Your Trace
You can use Application Insights to trace your triggers and verify if they match the cron expression.

### Need Assistance?
We at DevUP are experts in this area, and we use our service **Helium** to quickly scan the entire environment and retrieve the TLS versions per resource. We stay informed and will keep you updated on new TLS versions as they are released.

Reach out, and we can show you how to obtain this information in minutes and continuously.
