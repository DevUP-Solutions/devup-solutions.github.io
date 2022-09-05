---
layout: post
title: "Durable Functions: Utilize retries"
description: Azure Function comes with an extension called Durable Functions. | This post explains how to work with retires in order to build robust scalable Durable Functions.
date: 2022-09-29 08:00:00 +0200
categories: functions tips&tricks
tags: [Azure Functions, Integration, Tips and Tricks]
author: "Mattias Lögdberg"
comments: true
---

Azure Functions extension Durable Function is a powerfull extension to build long running well organized background jobs. In the Durable Functions extension we get alot of different things but the most important ones are the 3 different types we use when implenting Durable Functions. First we have thewe have the **Orchestration**, then we have the **Activity** and last we have **Entity** all of these has theire speciall abilites and sitations where they have the best fit. [Read more about the different types.](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-types-features-overview)

### Background
When building for scale we soon end up in managing retries, they are needed for many reasons the one we had lately is that the target api sometimes might start returning 429 or 503 responses due to high pressure. If and when this happens we need to back of a bit and retry again after either a set of time or a calculated time depending on how much informaton we get back from the api.

### How does it work
In order to get a good retry strategy we want to use the built in functionality in the Durable Functions framework.

In this case we are using an **Orchestration** function that runs for a given context, when it need's to call the api endpoint we do so via an **Activity** function.

Here is a sample to show you what’s happening. We create a simple trace snippet and add that to a new *Policy fragment*:

{% highlight xml %}
<fragment>
	<trace source="fragment">
        Trace from fragment simple sample
    </trace>
</fragment>
{% endhighlight %}

We can now add the above fragment to our *policies*.

{% highlight xml %}
<include-fragment fragment-id="demofragment" />
{% endhighlight %}


Adding this to the policy it will look like this:
{% highlight xml %}
<policies>
    <inbound>
        <base />
        <!-- other polices here-->
        <include-fragment fragment-id="demofragment" />
        <!-- other polices here-->
    </inbound>
    <backend>
        <forward-request timeout="300" buffer-request-body="true" />
    </backend>
    <outbound>
        <base />
        <!-- other polices here-->
        <include-fragment fragment-id="demofragment" />
        <!-- other polices here-->
    </outbound>
    <on-error>
        <base />
        <!-- other polices here-->
    </on-error>
</policies>
{% endhighlight %}

But at runtime it will be changed to:

{% highlight xml %}
<policies>
    <inbound>
        <base />
        <!-- other polices here-->
        <trace source="fragment">
            Trace from fragment simple sample
         </trace>
        <!-- other polices here-->
    </inbound>
    <backend>
        <forward-request timeout="300" buffer-request-body="true" />
    </backend>
    <outbound>
        <base />
        <!-- other polices here-->
        <trace source="fragment">
            Trace from fragment simple sample
        </trace>
        <!-- other polices here-->
    </outbound>
    <on-error>
        <base />
        <!-- other polices here-->
    </on-error>
</policies>
{% endhighlight %}

As you can see the code snippet in the *fragment* has now replaced the ´´´<include-fragment fragment-id="demofragment" />´´´ at all places. A change to the fragment will create a change in all places at next time it's loaded into the policy.

#### Doing more complex things
You can do anything in a fragment that you did in the original policy, just make sure to add variables and other dependencies in the policy before the fragment. As you saw in the merged policy above, during runtime it will merge into one policy that is executed.

### Why use it
Reusable snippets are always a good idea, just make sure to do them in a global thinking. Making them to small might be cumbersome since the list will be growing rapidly and the benefit of having a library of code snippets might be lost.


### Summary
We encounter situations all the time that makes us copy/paste code, fragments are a great way to standardise part of your policy and make development faster and more reliable.

Create a naming standard since the name will be the identifier and hard to change afterwards.