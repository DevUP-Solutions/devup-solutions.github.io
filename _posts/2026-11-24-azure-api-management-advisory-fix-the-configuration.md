---
layout: post
title: "Microsoft Fixed the Platform. The Configuration Is Still Ours."
description: "Microsoft's November advisory for Azure API Management Classic tiers names two conditions that decided who was exposed. Rotating the secrets closes the incident. It does not change why we were exposed."
date: 2026-11-24 09:00:00 +0100
categories: [Helium, Security]
tags: [Azure, API Management, Security, Key Vault, Managed Identity, Network Isolation, Governance]
author: "Mattias Lögdberg"
comments: true
---

For years, I have repeated the same advice in sessions, customer discussions, and our own work at DevUP:

> Static keys are bad. Move to identities.

On 22 November, Microsoft sent out a security advisory that reads like the reason why.

If you run Azure API Management on a Classic tier, the short version is this: rotate the credentials stored in it this week. Then fix the two settings that decided whether you were exposed.

The first part is urgent. The second part is the one that matters.

## What happened: the platform was fixed in a day

Everything in this section comes from Microsoft's own notification, sent by email to affected customers and published as a security advisory in Azure Service Health (tracking ID MZK9-GH0).

On 18 November 2026, Microsoft identified a security issue in the API Management Classic tiers. It could allow unauthorized access to certain management functionality, including secrets management, if the service was not configured according to Microsoft's [documented security best practices](https://learn.microsoft.com/azure/api-management/secure-api-management). Microsoft closed the access path on 19 November, with no action needed from customers. The v2 tiers were not affected.

That is a fast response, and it is Microsoft's side of the line.

But Microsoft is also clear about what the fix does not do. In its own words, the remediation "does not automatically invalidate credentials" that may have been accessed before it was deployed. And its investigation found that customers were exposed: an unauthorized party could have used the issue to access secrets stored in Classic instances before Microsoft found it.

Closing the door does not change what may already have left the room.

Microsoft lists what could have been accessed:

- Secrets stored in named values
- Client secrets for identity providers
- Client secrets for OpenID Connect providers
- Subscription keys
- Tokens used to call the direct management APIs

Each of those still works until somebody rotates it.

## Who was exposed: two conditions, both ours

Microsoft names two configurations. If you run a Classic tier in either of them, your resources may have been accessed:

- The instance lacks the network integration described in the [Network security](https://learn.microsoft.com/azure/api-management/secure-api-management#network-security) section of Microsoft's guidance.
- Secrets are stored in API Management instead of in Azure Key Vault and referenced from there, as the [Data protection](https://learn.microsoft.com/azure/api-management/secure-api-management#data-protection) section recommends.

Neither of these is a Microsoft setting. Both are decisions we made, or decisions we postponed.

I understand how they happen. Putting a secret straight into a named value is the quickest way to get an API working. Then it works, and nobody goes back. Network isolation costs money and effort, so it waits for the next project.

Nobody acted badly. The configuration simply stayed where the first deadline left it.

## This week: what Microsoft recommends

Microsoft's recommended actions, in its order:

1. Find your potentially impacted resources. They are listed under "Impacted Resources" on advisory MZK9-GH0 in Azure Service Health.
2. Immediately find and rotate the credentials and secrets stored in those instances, using Microsoft's [rotation guide](https://aka.ms/apimcredrotation).
3. Check the systems those secrets give access to for unexpected activity, and review your logs as far back as your records allow.
4. Consider migrating from the Classic tiers to v2, which was not affected.

One detail in the rotation guide is easy to miss. Updating the value in API Management does not revoke the old credential where it was issued.

So the order matters, and Microsoft spells it out: create the new credential at the issuer, update API Management, test, and then revoke the old one.

## A check we can all run today

Microsoft's guide includes a command that lists the named values in an instance and shows which secrets are backed by Key Vault:

```bash
az apim nv list --resource-group {rg} --service-name {apimName} \
  --query '[].{NamedValueId:name, Name:displayName, Secret:secret, KeyVaultBacked:!!(keyVault.secretIdentifier)}' \
  --output table
```

Every row where `Secret` is true and `KeyVaultBacked` is false is a secret stored in API Management itself.

That is the second condition in the advisory, as a list with names on it.

## Rotation is cleanup. Configuration is prevention.

Rotating secrets closes this incident.

It does not change why the instance was exposed.

If the new secret goes back into a plain named value, on an instance that is still open to the internet, we are exactly where we started. We have only changed the password.

The lasting fix is not my advice. It is the configuration in Microsoft's own [security guidance for API Management](https://learn.microsoft.com/azure/api-management/secure-api-management), the same document the advisory points to:

- Secrets in Key Vault, referenced from API Management through a managed identity.
- Network isolation, through a virtual network or a private endpoint with public access turned off.
- Managed identities instead of keys when API Management calls a backend.

## Key Vault is the minimum, not the goal

Microsoft's advisory draws the line at Key Vault. This is the one place where I go further than the advisory, so read it as my view and not Microsoft's.

In [Every AI Agent Needs an Identity — and an Owner]({% post_url 2026-10-01-every-ai-agent-needs-an-identity %}) I wrote that a protected static key is still a static key. It still needs to be rotated. It can still be shared. It can still be stolen.

The same is true here.

A secret in Key Vault would have kept an instance outside the second condition. A backend called with a managed identity would have had no secret to read in the first place.

> The goal is not to hide credentials better. The goal is to remove them wherever possible.

This becomes very concrete with AI. An API Management instance that calls Azure OpenAI with an API key usually keeps that key in a named value, which is the first item on Microsoft's list. With a managed identity, there is nothing stored to rotate.

## Where Helium fits, and where it does not

We built Helium to check this kind of configuration every day. So I want to be exact about what it does for this advisory.

For each API Management instance, Helium flags:

- secret named values that are not Key Vault references
- credentials and certificates held in API Management instead of in Key Vault
- instances without a managed identity
- instances that are publicly reachable, with no virtual network and no private endpoint
- AI backends called with an API key instead of a managed identity
- management API settings that still allow read-only access to secrets

Each finding names the resource and comes with remediation steps. When it is fixed, the next scan confirms it.

Helium does not detect whether a secret was read. It does not rotate credentials. Microsoft's guide and your own logs are still the answer to both.

What it does is keep the two conditions from this advisory visible every day, and not only in the week an advisory lands.

## We still own it

Cloud providers will keep finding and fixing issues in their platforms. Microsoft did that here, quickly.

Whether one of those issues becomes our incident depends on the configuration on our side of the line.

The platform was fixed in a day.

The configuration is still ours.

If you want to know where your API Management instances stand, start with the command above. If you want the whole environment checked, that is what our [Azure Health Check](https://www.devup.solutions/campaign/azure-health-check) is for.
