---
layout: post
title:  "APIM sending requests to LA with no payload!"
date:   2022-08-04 11:25:18 +0200
categories: apim issues
tags: [API Management, Integration, Logic Apps, Issues]
comments: true
---

Our Logic Apps that was behind API Management suddenly started to recieve empty payloads after our instance got the [July, 2022 release](https://github.com/Azure/API-Management/releases/tag/release-service-2022-07). We started an ticket and found out the following.

## Issue
When APIM recieves forwards a message to the backend it does so after process the *inbound policies* if it at this point hasen't read the full payload and are not sure that the **Content-Length** header equals the actual payload length. It will add a new header when sending the request to the backend **Transfer-Encoding** and here is where it starts to be problem. The [Logic App trigger dosen't support chunking](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-handle-large-messages), so it ends up triggering the run with an empty payload.

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
In orrder to solve this we need to make sure the payload is read to fully in APIM, this can easily be achieved with the following policy added in the **inbound** policy.

{% highlight xml %}
<set-body template="none">@{
return context.Request.Body.As<string>(preserveContent: true);
}</set-body>
{% endhighlight %}

I also got recomended from Microsoft Support to explicitly declare the forward request in the **backend** section aswell adding setting buffer for request body to true.

{% highlight xml %}
<forward-request timeout="300" buffer-request-body="true" />
{% endhighlight %}

The full policy will then look like this: (removed all other stuffs for simplicity)

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

After we added this no more issues where found.

## Summary
We encountered this on large messages sent, as we got from the Microsoft Support our problem started at 1000 byte's so we will from now on add this solution to all our api's that have potentially larger payload than 1000 bytes. But hopefully Logic App http triggers will support chunked messages in the future. 


## Helium - CH100200 Payload warning
As we found this issue we also toke the opportunity to add it to **Helium** to make sure all of our customers can get this warning and automated scans for all theire endpoints. To make sure they are not encountering the same issue.