---
layout: post
title: "Issue: Logic App recieves no payload from Api Management"
description: Logic App is not supporting chunked messages, in this case Api Management suddenly started to do send chunked messages. | This post explains why it happens and how we solved it. 
date: 2022-08-22 10:00:00 +0200
categories: logicapps issues
tags: [API Management, Integration, Logic Apps, Issues]
author: "Mattias Lögdberg"
comments: true
---

Our Logic Apps suddenly started to receive empty payloads from API Management after our APIM instance got the [July, 2022 release](https://github.com/Azure/API-Management/releases/tag/release-service-2022-07). We opened a ticket with Microsoft Support and found out the following:

### Issue
When API Management forwards a message to the backend it does so when reaching the backend section of the policies. When this section starts there is a short period of time where APIM reads the request body and if APIM is not done reading the body during this *period* it will (due to performance improvements) start sending chunked messages to the backend service. This is for most backend service a standard and good way, but since [Logic App trigger does not support chunking](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-handle-large-messages) this becomes a problem for us. Since the Logic App will start a run by the incoming request but since it dosen't support chunking the run will have a empty incoming payload.


This can be detected when the receiving service starts to get requests with the header Transfer-Encoding: chunked.

Example output of the trigger action on Logic Apps, with the **Transfer-Encoding** header and no **body** property.
{% highlight json %}
{
    "headers": {
        "Connection": "Keep-Alive",
        "Transfer-Encoding": "chunked",
        "Accept": "application/json",
        "Host": "prod-62.northeurope.logic.azure.com",
        "Request-Context": "appId=cid-v1:e91f2bbd-f887-4c87-a1f7-45c78f221b80",
        "Request-Id": "|8f1f9317-4dc8733f41b58527.8f1f9318_",
        "X-Forwarded-For": "{ip}}",
        "Content-Type": "application/json",
        "Content-Length": "0"
    }
}
{% endhighlight %}


### Solution
In order to solve this we need to make sure the entire request body is read in APi Management before the request is started to be sent to the backend. This is fortunally easily achieved with adding a read of the incoming body in the *inbound* policy section. Here is a simple example of how this could be achived, setting the body of the request for the backend to the full read incoming request from the caller:

{% highlight xml %}
<set-body template="none">@{
return context.Request.Body.As<string>(preserveContent: true);
}</set-body>
{% endhighlight %}

We where also recommended by Microsoft Support to explicitly declare the forward request in the backend section and set the buffer for the request body to true.

{% highlight xml %}
<forward-request timeout="300" buffer-request-body="true" />
{% endhighlight %}

The full policy will then look like this: (removed all the other stuffs for simplicity)

{% highlight xml %}
<policies>
    <inbound>
        <base />
        <!-- other polices here-->
        <set-body template="none">@{
        return context.Request.Body.As<string>(preserveContent: true);
        }</set-body>
        <!-- other polices here-->
    </inbound>
    <backend>
        <forward-request timeout="300" buffer-request-body="true" />
    </backend>
    <outbound>
        <base />
        <!-- other polices here-->
    </outbound>
    <on-error>
        <base />
        <!-- other polices here-->
    </on-error>
</policies>
{% endhighlight %}

When this fix was added the issues we had in Logic Apps where solved.

### Summary
We encountered this on large messages that where sent to us. But Microsoft Support informed us that our problems already started around 1000 kilobytes. So from now on we will add this fix to all our APIs that potentially will receive a large payload. Hopefully Logic Apps http trigger will support chunked messages in the future.