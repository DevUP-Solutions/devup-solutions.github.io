---
layout: post
title: "How to connect Event Grid to secured Function App"
description: Securing App Service with Authentication is getting more and more important here we go thur how to connect event grid to a protected App Service.
date: 2025-02-19 09:00:00 +0200
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

Define the necessary **App Registration** and **Service Principal** in Bicep. The Event Grid subscription requires an app registration so it can securely authenticate when sending events to the function. This ensures that only authorized sources can invoke the function, enhancing security and preventing unauthorized access.

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

- The Function App registration must include an app role, e.g., AzureEventGridSecureWebhookSubscriber.

Ensure you output the App ID for both the Function App app registration and the Event Grid Subscription app registration, as they will be used later.

## Step 3: Create and Manage Secrets for the Event Grid Subscription App

Use an Azure CLI script to reset the client secret without creating a new one on every deployment. This is necessary because the Event Grid subscription service principal requires a valid client secret to authenticate when creating the subscription. By resetting instead of recreating the secret, we avoid unnecessary credential rotations while ensuring security and seamless automation in DevOps.
```yaml
parameters:
  - name: egSubAppId
    type: string
  - name: egSubSecretName
    type: string
    default: "Your secret name"
  - name: serviceConnection
    type: string

steps:
- task: AzureCLI@2
  displayName: 'Create Secret for Event Grid Subscription App'
  inputs:
    azureSubscription: ${{ parameters.serviceConnection }}
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      echo "Creating secret for EG Subscription App..."
      EGSUBAPP_SECRET=$(az ad app credential reset --id ${{ parameters.egSubAppId }} --display-name ${{ parameters.egSubSecretName }} --years 1 --output tsv --query password)

      if [ -z "$EGSUBAPP_SECRET" ]; then
        echo "Failed to create secret for EG Subscription App"
        exit 1
      fi
      
      # Store the Event Grid Subscription App secret in a temporary file for later use
      echo "$EGSUBAPP_SECRET" > EGSUBAPP_SECRET.txt
```
> **Note:**
If using a hosted agent, the file will be deleted when the job finishes. No secret will be stored in DevOps; it is only used temporarily during execution and securely removed afterwards.

## Step 4: Assign Function App Role to the Event Grid Subscription App

A PowerShell script assigns the necessary role using the Microsoft Graph API. This role, which was created for the function app application in a previous step, is required to allow the Event Grid subscription service principal to authenticate and invoke the function securely. Without this role, Event Grid would not have the necessary permissions to send events to the function.

```yaml
parameters:
  - name: azureResourceManagerConnection
    type: string
  - name: functionAppId
    type: string
  - name: eventGridSubAppId
    type: string
  - name: functionAppdRoleName
    type: string
    default: "AzureEventGridSecureWebhookSubscriber"

steps:
- task: AzurePowerShell@5
  displayName: 'Assign Event Grid Subscription Role'
  inputs:
    azureSubscription: ${{ parameters.azureResourceManagerConnection }}
    ScriptType: InlineScript
    Inline: |
      # Function to convert a plain text string to SecureString
      function ConvertTo-SecureString {
          param (
              [string]$plainText
          )
          $secureString = New-Object System.Security.SecureString
          $plainText.ToCharArray() | ForEach-Object { $secureString.AppendChar($_) }
          return $secureString
      }

      $context = Get-AzContext
      $graphToken = [Microsoft.Azure.Commands.Common.Authentication.AzureSession]::Instance.AuthenticationFactory.Authenticate($context.Account, $context.Environment, $context.Tenant.Id.ToString(), $null, [Microsoft.Azure.Commands.Common.Authentication.ShowDialog]::Never, $null, "https://graph.microsoft.com").AccessToken
      $secureGraphToken = ConvertTo-SecureString -plainText $graphToken
      Connect-MgGraph -AccessToken $secureGraphToken
      $functionAppId = '${{parameters.functionAppId}}'
      $EventGridAppId = '${{parameters.eventGridSubAppId}}'
      $functionAppdRoleName = '${{parameters.functionAppdRoleName}}'
      
      # Get the Event Grid role ID
      Write-Host "Fetching Role ID for '$functionAppdRoleName'..."
      $functionApp = Get-MgServicePrincipal -Filter "appId eq '$functionAppId'"
      $functionAppRole = $functionApp.AppRoles | Where-Object { $_.DisplayName -eq $functionAppdRoleName }
      $EventSubscriptionWriterSpId = (Get-MgServicePrincipal -Filter "appId eq '$EventGridAppId'").Id
      $ServicePrincipalId = (Get-MgServicePrincipal -Filter "appId eq '$functionAppId'").Id

      if (-not $functionAppRole) {
          Write-Host "Error: Role '$functionAppdRoleName' not found!"
          exit 1
      }

      # Check if the role is already assigned
      $existingAssignment = Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $EventSubscriptionWriterSpId |
          Where-Object { $_.AppRoleId -eq $functionAppRole.Id }

      if ($existingAssignment) {
          Write-Host "Role '$functionAppdRoleName' is already assigned. Skipping..."
      } else {
          # Assign the role
          Write-Host "Assigning Role '$functionAppdRoleName' to Service Principal..."
          New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $EventSubscriptionWriterSpId `
              -PrincipalId $EventSubscriptionWriterSpId `
              -ResourceId $ServicePrincipalId `
              -AppRoleId $functionAppRole.Id
          Write-Host "Role assignment completed successfully!"
      }

    azurePowerShellVersion: LatestVersion
```

## Step 5: Create the Event Grid Subscription via CLI

The Event Grid subscription must be created using the service principal for authentication to ensure secure, role-based access control. This prevents unauthorized entities from creating or modifying subscriptions while allowing the service principal to authenticate and manage the subscription lifecycle programmatically.

```yaml
parameters:
  - name: egSubAppId
    type: string
  - name: tenantId
    type: string
  - name: resourceGroup
    type: string
  - name: topicName
    type: string
  - name: functionAppEndpoint
    type: string
  - name: functionAppId
    type: string
  - name: serviceConnection
    type: string
  - name: subscriptionName
    type: string

steps:
- task: AzureCLI@2
  displayName: 'Log in and Create Event Grid Subscription'
  inputs:
    azureSubscription: ${{ parameters.serviceConnection }}
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      echo "Checking if Event Grid Subscription '${{ parameters.subscriptionName }}' exists..."
      SUBSCRIPTION_EXISTS=$(az eventgrid topic event-subscription show \
        --name ${{ parameters.subscriptionName }} \
        --resource-group ${{ parameters.resourceGroup }} \
        --topic-name ${{ parameters.topicName }} \
        --query "provisioningState" --output tsv 2>/dev/null || echo "NotFound")

      if [[ "$SUBSCRIPTION_EXISTS" == "Succeeded" || "$SUBSCRIPTION_EXISTS" == "Updating" ]]; then
        echo "Event Grid Subscription already exists. Skipping creation."
        exit 0
      fi

      echo "Retrieving the EG Sub App secret from the temporary file..."
      EGSUBAPP_SECRET=$(cat EGSUBAPP_SECRET.txt)

      if [ -z "$EGSUBAPP_SECRET" ]; then
        echo "Failed to retrieve Event Grid Sub App secret"
        exit 1
      fi

      echo "Logging into Azure using Event Grid Sub App..."
      az login --service-principal -u ${{ parameters.egSubAppId }} -p "$EGSUBAPP_SECRET" --tenant ${{ parameters.tenantId }}

      echo "Creating Event Grid Subscription..."
      az eventgrid topic event-subscription create \
        --name ${{ parameters.subscriptionName }} \
        --resource-group ${{ parameters.resourceGroup }} \
        --topic-name ${{ parameters.topicName }} \
        --endpoint ${{ parameters.functionAppEndpoint }} \
        --azure-active-directory-tenant-id ${{ parameters.tenantId }} \
        --azure-active-directory-application-id-or-uri ${{ parameters.functionAppId }} \
        --endpoint-type webhook

      echo "Event Grid subscription created successfully!"
```

## Summary

By following these steps, you can securely subscribe an Azure Event Grid topic to an Easy Auth-protected Azure Function within an automated DevOps pipeline. This approach ensures that only authorized Event Grid subscriptions can send events to your function, reducing the risk of unauthorized access while maintaining security best practices. By automating this process, you not only streamline deployment but also enforce consistency in role assignments and authentication configurations across different environments.



### If I need some assistance?
We at DevUP are experts in this area, and we use our service **Helium** to quickly scan the whole environment and get the TLS versions per resource, we know and will let you know when new versions of TLS is released.

Reach out, and we can show you how to get this information in minutes and continuously.