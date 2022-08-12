---
layout: post
title:  "Issue: Logic App recieves no payload from Api Management"
date:   2022-08-12 12:00:00 +0200
categories: logicapps issues
tags: [API Management, Integration, Logic Apps, Issues]
comments: true
---

Our Logic Apps suddenly started to receive empty payloads from API Management after our APIM instance got the [July, 2022 release](https://github.com/Azure/API-Management/releases/tag/release-service-2022-07). We opened a ticket with Microsoft Support and found out the following:

## Issue
When API Management forwards a message to the backend it does so when reaching the backend section of the policies. When this section starts there is a short period of time where APIM reads the request body. If APIM fails to read the entire body in this “short period” it will start to send chunked messages (performance improvement) to the backend service. This can be detected when the receiving service starts to get requests with the header Transfer-Encoding: chunked. This is where it starts to be a problem if you have Logic Apps as the receiving service. [Logic App trigger does not support chunking](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-handle-large-messages), so it ends up triggering the run as expected but with an empty payload.

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


## Solution
In order to solve this we need to make sure the entire request body is read in APIM. This is easily achieved with the following policy added in the inbound policy.

{% highlight xml %}
<set-body template="none">@{
return context.Request.Body.As<string>(preserveContent: true);
}</set-body>
{% endhighlight %}

I was also recommended by Microsoft Support to explicitly declare the forward request in the backend section and set the buffer for the request body to true.

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

## Summary
We encountered this on large messages that where sent to us. But Microsoft Support informed us that our problems already started around 1000 kilobytes. So from now on we will add this fix to all our APIs that potentially will receive a large payload. Hopefully Logic Apps http trigger will support chunked messages in the future.


## Helium - CH100200 Payload warning
When we found this issue we also took the opportunity to add this control to Helium to make sure all of our customers will not encounter the same issue.
