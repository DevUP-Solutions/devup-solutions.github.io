---
layout: post
title: "App Configuration in Azure Integration Services"
description: App Configuration for Functions is finally in preview! More flexibility and easier usage is now on the horizon.  | This post explains an overview of how a setup could look like and how it works.
date: 2023-02-16 09:00:00 +0200
categories: appconfiguration maintainability
tags: [App Configuration, Integration, Maintainability ]
author: "Mattias Lögdberg"
comments: true
---

App Configuration is a central place to store shared configuration values, like URL’s for a service, domains, organisations or other non secret parameters. This is something that is very important in a microservice setup where multiple services connect to the same source or target.

There are .NET and java frameworks for connecting to App Configuration but Web Apps/Azure Functions runtime has been lacking app config reference support. This makes it much easier to start using App Configuration and also brings the capabilities to Logic App Standard*.

### Background
App Configuration has been a big part in centralizing settings that are used by several microservices. These settings have previously been used through code, the code is easily implemented when using the **Microsoft.Extensions.Configuration.AzureAppConfiguration** (NuGet package)[https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureAppConfiguration/] but it makes migration harder and forces us to manage caching of the values collected.

### How does it work
Settings are stored inside Azure App Configuration and can be read by multiple sources, whenever a change is done in App Configuration the next time a source refreshes it's data the update will be applied. 

![App Configuration Overview](/assets/images/2023/february/appconfigoverview.jpg)

With references we can skip the extra code needed for fetching the setting from the App configuration instance and only do a "normal" fetch of a value from App Settings, such as: 
{% highlight code %}
var value = Environment.GetEnvironmentVariable("settingstest");
{% endhighlight %}

And only update our Application settings with the following value:
{% highlight code %}
@Microsoft.AppConfiguration(Endpoint={appconfigurationEndpoint}; Key={appconfigKeyValue})
{% endhighlight %}

Here is an example of how it could look like:
{% highlight code %}
{
    "name": "settingstest",
    "value": "@Microsoft.AppConfiguration(Endpoint=https://devuptest.azconfig.io; Key=firstdata)​",
    "slotSetting": false
}
{% endhighlight %}

Using references also makes it easier to do local debugging  as we can add the settings to our local.settings.json as usual. 


### Improvements
Primary improvements found so far is that Api Management and Logic App Consumption would be integrated "out of box" as well. So we can share settings across all processing services.

### Why use it
Centralize settings in one place in a micro service landscape helps preventing issues when changes are done to settings. We have seen so many cases where one or a few places were forgotten and strange problems has occurred as a result. Creative workarounds have of course been found, such as using key vault but that will just compromise the secrets stored there.

### Summary
We highly recommend using App Configuration to share settings values across micro services since it will give you a better overview on settings and making sure the updates are reflected at all places. It's a good complement to Key Vault and sdk's are available in many languages, se link bellow for more information. There are also more to the product than storing settings, for example feature flag which could help in sharing feature flag functionality over your micro service landscape as well, [read more.](https://learn.microsoft.com/en-us/azure/azure-app-configuration/overview)
