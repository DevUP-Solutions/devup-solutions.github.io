---
layout: post
title: "Api Management: Policy Fragments, a powerfull friend"
description: Api Management released Policy Fragments earlier this year and i's been little talk about the. | This post explains why Policy Fragments are so great to use. 
date: 2022-09-01 08:00:00 +0200
categories: apimanagement tips&tricks
tags: [API Management, Integration, Tips and Tricks]
author: "Mattias Lögdberg"
comments: true
---

Api Management released a new feature earlier this year called Policy fragments. In this post we will go through what it is and show why you should start using them!

### Background
In API Management we have policies that we can configure to manage incoming requests from API consumers. Through policies we can manage request before forwarding them to backend systems and before returning the response to the consumer. This gives us flexibility and power to do a lot of things. The most common things we add are increased security, manage of routing to backend and transformation of incoming and response messages.

Even if we have a layered approach in Api Management where we can add policies on instance -> product -> api -> operation level (these will be combined starting from operation and moving upward adding the policy instead of the```<base/>``` tag. We still end up with a lot of duplicate code and copy paste situations.

Let’s say you want a specific configuration for 2 of 3 operations in an API. This usually ends up with copy pasting the same code snippet and here is where Policy fragments comes in and shines.

### How does it work
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