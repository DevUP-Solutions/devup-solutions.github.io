---
layout: post
title: "Api Management: Authroizations"
description: Api Management released a preview of Authorizations earlier this year. | This post explains why Authorizations is a neat way of handling authrorization for all backends protected by OAuth.
date: 2022-09-30 14:00:00 +0200
categories: apimanagement tips&tricks
tags: [API Management, Integration, Tips and Tricks]
author: "Mattias Lögdberg"
comments: true
---

Api Management released a new feature earlier this year called Ahtorizations it's still in *preview*. In this post we will go through what it is and show why you should start using it!

### Background
In API Management we want to be able to connect to many systems in order to provide a good api for our consumers. These systems often have different ways of authentication, the ones that are protected by Oauth2 have now gotten a much better experince, [read details on Authorizations overview](https://learn.microsoft.com/en-us/azure/api-management/authorizations-overview).

The most important part is that it simplifies the policy tremendously.

### How does it work
The Api Management team have built standard code for managing authroization and state for us, which removes a lot of customization previously needed and makes it easier to separate code from authroization, reuse and manage.

This image below is taken from the [Authroizations Ovreview](https://learn.microsoft.com/en-us/azure/api-management/authorizations-overview) and in a very good way describes how it works. Simply described the *Authroization* resource is hanling all the complexity to create a token, cache the token and before a new call validate if the token still is valid.


![Get Token path](https://learn.microsoft.com/en-us/azure/api-management/media/authorizations-overview/get-token-for-backend.svg)


[Here](https://learn.microsoft.com/en-us/azure/api-management/authorizations-how-to) is a great guide to get started.


#### Sample of change
Let's take one of our implementations to Dynamics 365 FNO as a sample.
This is how the *policy* looked like before:

{% highlight xml %}
<policies>
    <inbound>
        <base />
        <cache-lookup-value key="@("bearerToken")" variable-name="bearerToken" />
        <choose>
            <when condition="@(!context.Variables.ContainsKey("bearerToken"))">
                <send-request ignore-error="true" timeout="20" response-variable-name="accessTokenResult" mode="new">
                    <set-url>https://login.microsoftonline.com/{{tenantiId}}}/oauth2/token</set-url>
                    <set-method>POST</set-method>
                    <set-header name="Content-Type" exists-action="override">
                        <value>application/x-www-form-urlencoded</value>
                    </set-header>
                    <set-body>@{
                    var uriStr = context.Api.ServiceUrl.ToString();
              return "client_id={{Secret-ClientId}}&resource="+uriStr.Remove(uriStr.Length - 1)+"&client_secret={{Secret-ClientSecret}}&grant_type=client_credentials";
                    }</set-body>
                </send-request>
                <set-variable name="accessToken" value="@(((IResponse)context.Variables["accessTokenResult"]).Body.As<JObject>())" />
                <set-variable name="bearerToken" value="@((string)((JObject)context.Variables["accessToken"])["access_token"])" />
                <set-variable name="tokenDurationSeconds" value="@((int)((JObject)context.Variables["accessToken"])["expires_in"])" />
                <cache-store-value key="bearerToken" value="@((string)context.Variables["bearerToken"])" duration="@((int)context.Variables["tokenDurationSeconds"])" />
            </when>
        </choose>
        <set-header name="Authorization" exists-action="override">
            <value>@("Bearer " + (string)context.Variables["bearerToken"])</value>
        </set-header>
        <rewrite-uri template="/service/route" copy-unmatched-params="true" />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
{% endhighlight %}

And here is after:

{% highlight xml %}
<policies>
    <inbound>
        <base />
        <get-authorization-context provider-id="d365-dev" authorization-id="d365-dev-2" context-variable-name="auth-context" identity-type="managed" ignore-error="false" />
        <set-header name="Authorization" exists-action="override">
            <value>@("Bearer " + ((Authorization)context.Variables.GetValueOrDefault("auth-context"))?.AccessToken)</value>
        </set-header>       
        <rewrite-uri template="/service/route" copy-unmatched-params="true" />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
{% endhighlight %}

As you can see the policy is much easier to understand and work with. We get dedicated resources in *API Management* to work with and that also makes the deployment much better and easier. The authorzation handles the renew and caching of the token that we previously needed to manage ourselves.

### Improvments
My primary wish for improvements so far is that **Key Vault** will be integrated into the process. Since it not at the moment it becomes a bit annoying when i.e. working with grant type **Client credentials** since we don't want these credentials stored and managed manually in multiple places.

### Why use it
A first class resource manageing the token state to your provider, simplifying the process and makes our policies more readable. Also it prevent's a few basic errors that we often encounter the first times setting things up.

But the biggest part is that we now can separate the policy from the authorization, wich make using **Authorizations** a great separation of concern setup.

### Summary
As described above the *Authorization* is a great addition to *API Management*. Helping us keep authroization to backend services separated from policies so the policy are kept slimmer and authroization are managed by itself. Improves maintainabillity both on updates on the authorization and also by keeping our policies slimmer and easier to read.
