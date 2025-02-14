---
layout: post
title: "How to connect Event Grid to secured Function App"
description: Securing App Service with Authentication is getting more and more important here we go thur how to connect event grid to a protected App Service.
date: 2025-02-17 09:00:00 +0200
categories: Security Authentication
tags: [EntraID, Security, Authentication]
author: "Jonas Gaverus"
comments: true
---

# Creating an Event Grid Topic Subscription to a Secure (Easy Auth) Azure Function via DevOps

When deploying an Azure Function with Easy Auth enabled, setting up a secure Event Grid subscription can be challenging. This guide walks you through the process of configuring your function, creating the necessary Azure resources, and automating the setup using Azure DevOps.

## Summary of Steps

1. **Create an Azure Function with an HTTP Trigger** – Ensure your function handles the Event Grid subscription handshake.
2. **Create Function App and Event Subscription App Registrations via Bicep** – Define the required app registrations and service principals.
3. **Manage Secrets for the Event Grid Subscription App** – Reset the client secret securely without creating a new one on every deployment.
4. **Assign Function App Role to the Event Grid Subscription App** – Grant the necessary permissions for secure authentication.
5. **Create the Event Grid Subscription via CLI** – Use the service principal to authenticate and establish the subscription securely.

By following these steps, you ensure a secure and automated deployment of Event Grid subscriptions to an Easy Auth-protected Azure Function.

## Prerequisites

Ensure that your Azure DevOps service connection service principal has at least the following permissions. These permissions are required to manage application registrations and role assignments needed for secure authentication and authorization when setting up the Event Grid subscription:

- `Application.ReadWrite.All`
- `AppRoleAssignment.ReadWrite`
- `Directory.Read.All`

### Important Notes

- **This will not work with an Azure Function Event Grid trigger.** You must use an **HTTP trigger** instead.
- If you are using **.NET 8**, update your code to use `CloudEvents` instead of `EventGridEvent`.

## Step 1: Create an Azure Function with an HTTP Trigger

Your function must include code to handle the Event Grid subscription handshake, which is required for Event Grid to verify the endpoint before sending events. Without this handshake, the subscription creation process will fail. Below is an example implementation in C#:

```csharp
// Read the request body
string requestBody;
using (StreamReader reader = new StreamReader(req.Body))
{
    requestBody = await reader.ReadToEndAsync();
}

// Deserialize the request body into an EventGridEvent
var eventGridEvents = JsonSerializer.Deserialize<EventGridEvent[]>(requestBody);
if (eventGridEvents == null || eventGridEvents.Length == 0)
{
    _logger.LogError("Failed to deserialize event grid event");
    return new BadRequestObjectResult("Invalid event grid event");
}

foreach (var eventGridEvent in eventGridEvents)
{
    _logger.LogInformation("Event type: {type}, Event subject: {subject}", eventGridEvent.EventType, eventGridEvent.Subject);
    
    if (eventGridEvent.TryGetSystemEventData(out object eventData))
    {
        if (eventData is SubscriptionValidationEventData subscriptionValidationEventData)
        {
            _logger.LogInformation($"Got SubscriptionValidation event data, validation code: {subscriptionValidationEventData.ValidationCode}, topic: {eventGridEvent.Topic}");
            var responseData = new { ValidationResponse = subscriptionValidationEventData.ValidationCode };
            return new OkObjectResult(responseData);
        }
    }

    // Your custom event handling logic here
}
```

## Step 2: Create Function App and Event Subscription App Registrations via Bicep

Define the necessary **App Registration** and **Service Principal** in Bicep:

```bicep
extension 'br:mcr.microsoft.com/bicep/extensions/microsoftgraph/v1.0:0.1.8-preview'

param appName string
param uniqueName string
param uniqueString string
param eventGridSubscriptionAppDisplayname

resource adApp 'Microsoft.Graph/applications@v1.0' = {
  description: 'Backend for ${appName} instance'
  displayName: appName
  tags: [
    'DevUP was here'
  ]
  uniqueName: uniqueName
  signInAudience: 'AzureADMyOrg'
  appRoles: [
        {
      allowedMemberTypes: [
        'User'
        'Application'
      ]
      description: 'Azure Event Grid Role'
      displayName: 'AzureEventGridSecureWebhookSubscriber'
      id: '<set a static guid>'
      isEnabled: true
      value: 'AzureEventGridSecureWebhookSubscriber'
    }
  ]
}

var identiierUri = 'api://${adApp.appId}'

resource adAppIdentifier 'Microsoft.Graph/applications@v1.0' = {
  identifierUris: [
    identiierUri
  ]
  uniqueName: uniqueName
  displayName: appName
}

resource adAppServicePrincipal 'Microsoft.Graph/servicePrincipals@v1.0' = {
  displayName: appName
  appId: adApp.appId
}

resource EGSubAdApp 'Microsoft.Graph/applications@v1.0' = {
  displayName: eventGridSubscriptionAppDisplayname
  description: 'Ad app for event grid subscription'
  uniqueName: '${uniqueName}-EGSub'
}

resource EGSubServicePrincipal 'Microsoft.Graph/servicePrincipals@v1.0' = {
  displayName: eventGridSubscriptionAppDisplayname
  appId: EGSubAdApp.appId
}

output adAppId string = adApp.appId
output egSubAppId string = EGSubAdApp.appId
```

...

**Summary:**

By following these steps, you can securely subscribe an Azure Event Grid topic to an Easy Auth-protected Azure Function within an automated DevOps pipeline. This approach ensures that only authorized Event Grid subscriptions can send events to your function, reducing the risk of unauthorized access while maintaining security best practices. By automating this process, you not only streamline deployment but also enforce consistency in role assignments and authentication configurations across different environments.


### If I need some assistance?
We at DevUP are experts in this area, and we use our service **Helium** to quickly scan the whole environment and get the TLS versions per resource, we know and will let you know when new versions of TLS is released.

Reach out, and we can show you how to get this information in minutes and continuously.