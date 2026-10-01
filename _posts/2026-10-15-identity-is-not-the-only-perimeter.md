---
layout: post
title: "Identity Is Not the Only Perimeter"
description: "An agent can have the right identity and still send data somewhere it should not. What building Helium's MCP server taught us about access, tools, and the boundaries between them."
date: 2026-10-15 09:00:00 +0200
categories: [Helium, AI]
tags: [AI, Security, Azure, MCP, Network Security Perimeter, Microsoft Foundry, API Management, Governance]
author: "Mattias Lögdberg"
image: /assets/images/2026/agent-boundaries-hero.png
comments: true
---

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/agent-boundaries-hero-mobile.svg">
  <img src="/assets/images/2026/agent-boundaries-hero.svg" alt="An invoice agent reaches approved services through controlled paths. An approved MCP server forwards data to an attacker endpoint. Identity is not the only perimeter. Every path needs a boundary.">
</picture>

This is the third of four articles in this series. In [Every AI Agent Needs an Identity — and an Owner]({% post_url 2026-10-01-every-ai-agent-needs-an-identity %}), I argued that every agent needs its own identity, a clear authority model, and a named owner.

Identity gives us the context. Boundaries define where the agent can act. Validation tells us what actually happened.

This one is about boundaries: where an agent can connect, which tools it can use, and what data it can send through those connections.

Earlier this year we built an MCP server for Helium. It runs as an Azure Functions app, users sign in with their Entra account, and an AI assistant can ask about the Azure environments that user already has access to. It is available to every Helium customer today.

We made two deliberate choices. The tools are read-only. And connecting through MCP gives the user no more access to customer data than connecting through the portal.

That covers what our server can access and what its tools can do.

But what happens after the answer reaches the client?

## Incident: Every action was allowed

Microsoft's incident responders [describe an attack pattern against an invoice-processing agent](https://www.microsoft.com/en-us/security/blog/2026/06/30/securing-ai-agents-ai-tools-move-from-reading-acting/). The scenario illustrates techniques observed against enterprise agents in 2026.

An attacker changed the description of a tool on an approved third-party MCP server. The instruction directed the agent to collect a summary of unpaid invoices and include it in a normal tool call. The server returned a plausible answer and forwarded the summary to an attacker-controlled endpoint.

The tool was approved. The query used the analyst's permissions. The destination was allowlisted.

The data still left.

Perhaps the permissions were broader than the task required. That is not the lesson. Staying within a user's permissions and an approved destination list is still not enough to prevent unwanted disclosure.

Approving a server does not mean approving every piece of data an agent might send to it.

## Access: One model, every channel

An MCP server is another way into customer data. We need to distinguish two questions: who can connect, and what they can access once connected.

For Helium, connecting requires an Entra sign-in. Data access is then enforced by the same backend API the portal uses.

The MCP server exchanges the user's token for a backend token on their behalf. It does not use a separate, broadly privileged identity to read customer data. The backend decides which environments the user can access.

> The channel changes. The user's access should not.

It is easy to take a shortcut here: give the server broad read access, then filter the results inside each tool. Now there are two places deciding what a user may see. They both need to stay correct as permissions, tools, and customer environments change.

For a server acting on behalf of a user, we want that user's authority enforced at the backend. The same rule should hold whether the request comes from a portal, an API client, or an AI assistant.

Read-only adds another boundary. Our tools cannot change a customer's environment through Helium.

They can still return findings, resource names, failed checks, and remediation guidance. Once those answers reach the client, they can become part of a summary, another tool call, or a request to an external service.

Read-only limits what our server can do. It does not control what happens to the answer afterwards.

We enforce access in Helium. The customer governs the client and the other tools connected to it. Both parts need a clear owner and an understood boundary.

## Paths: Follow the complete flow

In the [governance article]({% post_url 2026-09-23-governance-needs-to-move-at-the-same-speed-as-ai %}), I mentioned a [DevUP Talks conversation with Simon Wåhlin](https://youtu.be/-eAswM6jzxA) about AI solving a connectivity problem by removing the firewall that blocked it.

Here, the problem is different. A boundary can remain in place while an allowed connection carries data somewhere it should not go.

Take the same agent from the identity article. It reads documents from Azure Storage, calls an MCP server, triggers a Logic App, and updates a customer system.

Last time, we asked which identity and permissions were used on each connection. Now we also need to ask what can pass through it.

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/agent-boundaries-chain-mobile.svg">
  <img src="/assets/images/2026/agent-boundaries-chain.svg" alt="An AI agent connected to Azure Storage, an MCP server, a Logic App, and a customer system. Each connection has an identity and a boundary. Controls depend on the resource and runtime.">
</picture>

Which data can the agent retrieve? Which tool operations are available? Can a tool accept arbitrary text, a destination URL, or an attachment? What can the receiving service do with that information?

These are integration questions. Each component can behave as configured while the complete flow does something we never intended.

## Controls: Azure covers different parts

Azure provides several useful boundary controls. We need to know which part of the flow each one actually covers.

| Boundary | What it helps control | What to verify |
| --- | --- | --- |
| Data resources | Network Security Perimeter controls public access for supported PaaS resources. | Resource support, access mode, explicit rules, and private access paths. |
| Agent runtime | Virtual network isolation and applicable egress controls restrict outbound connectivity. | The runtime's outbound configuration, not only its inbound private endpoint. |
| MCP tools | An API Management gateway can apply policies to routed requests. | Which tools actually use the gateway and which still connect directly. |

[Network Security Perimeter](https://learn.microsoft.com/en-us/azure/private-link/network-security-perimeter-concepts) is a logical boundary for supported PaaS services, including Storage, Key Vault, AI Search, and Foundry. In enforced mode it restricts public traffic, with explicit exceptions. It is not an egress firewall for the clients that read from them.

For [Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/networking-options), inbound and outbound networking are separate decisions. Adding a private endpoint while keeping public egress does not isolate the agent's outbound connections. [Hosted-agent egress controls](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/add-hosted-agent-guardrails) can restrict destinations, but are currently in preview and apply specifically to hosted agents.

[Foundry's MCP gateway routing](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/governance) is also in preview. It applies to eligible new MCP tools created in the portal after the gateway is connected. Existing tools are not automatically rerouted; managed OAuth and code-first MCP tools are among the exclusions.

And [some Foundry tools still use public endpoints](https://learn.microsoft.com/en-us/azure/foundry/how-to/configure-private-link#agent-tools-with-network-isolation) in network-isolated setups, including Bing grounding, web search, and SharePoint grounding. If the requirement is entirely private connectivity, those tools need an explicit decision.

The useful question is not whether we enabled network isolation. It is whether the paths this agent actually uses meet our requirements.

## Payload: An approved destination still needs limits

A network rule can restrict where an agent connects. It cannot, by itself, decide whether an invoice summary belongs in a particular tool call.

That needs another layer: scoped operations, validated parameters, and controls on the data being sent. Sensitive sharing or high-impact actions may also need human approval.

Microsoft's invoice example recommends inspecting outgoing tool parameters with data loss prevention controls and requiring approval for sensitive actions. The exact implementation depends on the platform, but the architectural question remains the same:

> Is this tool allowed to receive this data for this task?

A gateway is a place to enforce controls. Its presence alone does not answer that question. Neither does a successful sign-in.

## Descriptions: The wording changed the behavior

We saw the influence of tool descriptions in a much less dramatic way when building our own server.

Assistants kept choosing an environment-wide findings tool for questions about a single resource, then calling a detail tool repeatedly to assemble the answer. We already had a tool designed to return the resource's compliance information.

We rewrote the descriptions to make that choice clearer. That corrected the tool selection without changing the implementation.

The wording was part of the behavior we were shipping.

That is why tool metadata belongs in change review. A description can influence which operation an agent chooses and what it sends to that operation. The invoice attack used the same mechanism with a very different intention.

In our server, a repository catalog records which tools exist. A test checks that the registered tools match it. That protects the inventory. The description wording still needs review in the pull request.

Those are different checks. Knowing that a tool exists does not tell us whether its instructions are appropriate.

## Start: With the paths you actually use

Before tightening an allowlist, observe representative workloads. Network Security Perimeter has transition mode. Hosted-agent egress controls have audit mode for observing would-be denials before enforcing them.

Tools may call other hosts. SDKs and package managers may follow redirects.

The real dependency list can be longer than the one we drew.

I would start here:

1. **Map the complete flow.** Include the agent runtime, client, data sources, tools, downstream services, and destinations for results. Give each part an owner.
2. **Keep authorization consistent.** For delegated access, enforce the user's authority at the backend through every channel.
3. **Restrict connections and operations.** Allow the destinations and tools the task needs, then limit what those tools can do.
4. **Control the payload.** Validate parameters, minimize returned data, and inspect or approve sensitive sharing where required.
5. **Review tool changes.** Treat descriptions, schemas, publishers, and connection settings as changes to a production dependency.
6. **Observe, enforce, and test.** Check actual traffic and verify that prohibited operations and destinations are blocked.
7. **Check again after changes.** A new tool or a troubleshooting exception can create a path the original controls never covered.

## Drift: Boundaries drift too

A public endpoint is enabled for troubleshooting and never disabled. A new tool connects directly instead of through the gateway. An exception survives long after the reason for it disappeared.

This is why boundaries belong in continuous governance. They need to be followed throughout the workload's lifecycle.

It is the same loop we use when thinking about Azure governance in Helium:

**Discover → Understand → Prioritize → Improve → Verify**

A public endpoint is a finding. Understanding the resource's purpose, exposure, and available controls helps us decide whether it should be a priority.

Helium's role is to help teams understand their Azure environment and decide what to improve. Its MCP server makes that information available to an assistant under the user's existing access. Governing that assistant's runtime, tools, and subsequent use of the data is another part of the complete solution.

## Accountability: So, who owns the boundary?

We still do.

For our MCP server, we own the access enforcement, the operations we expose, and the information we return. Customers own the decisions about the clients and other tools they connect.

We need to make those responsibilities explicit. A flow crossing several systems should not become a flow nobody owns.

A boundary limits what could happen. To understand what did happen, we need evidence: which actor called which tool, with what parameters, under whose authority, and where the result went.

That is where the final article in this series begins. After identity and boundaries, we return to validation: what could the agent do, what did it do, and can we prove the complete chain?

---

**Want a clearer view of exposure and security configuration across your Azure environment?**

[Learn more about DevUP Helium](https://www.devup.solutions/) or [contact Mattias](mailto:mattias@devup.solutions).
