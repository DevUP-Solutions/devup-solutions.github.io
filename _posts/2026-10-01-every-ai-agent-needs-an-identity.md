---
layout: post
title: "Every AI Agent Needs an Identity — and an Owner"
description: "Replacing static keys with identities is still the right advice, but AI agents raise the stakes. Every agent needs its own identity, a clear authority model, least-privilege access, and a named owner."
date: 2026-10-01 09:00:00 +0200
categories: [Helium, AI]
tags: [AI, Security, Azure, Microsoft Entra, Agent ID, Managed Identity, Workload Identity, Governance]
author: "Mattias Lögdberg"
comments: true
---

In the previous article, [Who Owns the Architecture When AI Writes the Code?]({% post_url 2026-09-07-who-owns-the-architecture-when-ai-writes-the-code %}), I asked what happens when AI accelerates delivery faster than governance can follow.

My answer was simple:

> We still own it.

But ownership requires control. And control starts with knowing who—or what—is acting inside our systems.

This is the second of four articles in this series.

Identity gives us the context. Boundaries define where the agent can act. Validation tells us what actually happened.

This one is about identity.

For years, I have repeated the same advice in sessions, customer discussions, and our own work at DevUP:

> Static keys are bad. Move to identities.

That advice is still correct.

But it is no longer enough.

Applications used to authenticate, follow predefined logic, and call the services we had explicitly connected to them. AI agents can choose tools, combine information, call other agents, and act with a degree of autonomy.

The identity question is no longer only whether an application can connect. It is what a non-human actor can decide to do once connected.

In a recent DevUP Talks conversation with Markus Lintuala, he described the change very well:

> Our newest users are not humans. They are machines, and they work at machine speed.

That is the part I think many organizations are still underestimating.

## Credentials: Static keys were already a problem

Static credentials have never been a good foundation for cloud security.

They are copied into configuration, shared between services, forgotten in deployment pipelines, and sometimes left active long after the original integration has disappeared. When several workloads use the same key, it also becomes difficult to answer a very basic question:

> Who actually performed this action?

Moving a key from an application setting into Azure Key Vault is an improvement. It reduces the risk of exposing the credential in code or plain-text configuration.

But a protected static key is still a static key.

It still needs to be rotated. It can still be shared. It can still be stolen and used by something other than the workload it was intended for.

Where Azure services support identity-based authentication, the stronger pattern is to avoid storing the credential in the first place:

- Use managed identities for workloads running in Azure.
- Use workload identity federation for workloads running outside Azure or across platforms.
- Use Microsoft Entra Agent ID for agents on platforms that support the new agent identity model.
- Use Key Vault and frequent rotation only when a static secret cannot yet be avoided.

The goal is not simply to hide credentials better.

The goal is to remove them wherever possible.

Microsoft describes the same direction in its guidance for [workload identity federation](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation) and for [securing Azure MCP Server deployments](https://learn.microsoft.com/en-us/azure/developer/azure-mcp-server/security).

## Blast radius: AI makes it bigger and faster

AI agents do not create the need for least privilege. We needed that long before generative AI.

What changes is the speed, scale, and unpredictability of the actor using the permission.

This becomes very concrete in integration environments. An agent rarely touches only one isolated system. It can become the link between storage, APIs, workflows, messaging, and business systems.

Imagine an AI agent that:

- Reads documents from Azure Storage
- Calls an MCP server
- Triggers a Logic App
- Updates a customer system

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/agent-identity-chain-mobile.svg">
  <img src="/assets/images/2026/agent-identity-chain.svg" alt="An AI agent connected to Azure Storage, an MCP server, a Logic App, and a customer system, with an identity and permission marker on every connection">
</picture>

What looks like one agent permission can quickly become a cross-system chain of authority.

Each individual permission may look reasonable. The risk appears when the agent can combine them.

A misleading instruction, compromised tool, poisoned tool response, or simply an unexpected decision can turn several individually acceptable permissions into a much larger action chain. The agent can repeat that chain faster than a human and across far more data.

> Every agent identity defines a potential blast radius.

If the identity has broad access, the agent has broad access. If several agents share the same identity, we lose both isolation and traceability. If the agent calls a tool that uses an even more privileged backend identity, we may also create a confused deputy: a low-privileged caller borrowing the authority of a much more powerful service.

Authentication alone does not solve any of those problems.

## Identity: Every agent needs its own

Microsoft Entra Agent ID introduces identities created specifically for AI agents, together with blueprints for applying common policies and lifecycle controls. Microsoft’s current [Agent ID best practices](https://learn.microsoft.com/en-us/entra/agent-id/best-practices-agent-id) recommend a unique identity for each agent instance, with an assigned sponsor and owner.

That distinction matters.

A production agent should not disappear behind a shared application registration or a generic integration identity. It needs to be visible as an actor in its own right.

That gives us the ability to:

- Assign only the permissions that agent needs
- Trace its actions independently
- Review its access without affecting unrelated agents
- Disable it without breaking every other workload
- Tie its lifecycle to a known business purpose

There can be practical reasons to use a shared project identity during early development. At the time of writing, Microsoft Foundry does this for unpublished agents. But as an agent moves toward integration testing or production, its permissions, audit trail, and lifecycle need to become explicit. Foundry’s [agent identity model](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/agent-identity) supports distinct identities for published agents.

Sharing an identity may reduce administrative work today.

It also increases the blast radius you will need to explain tomorrow.

## Authority: Decide whose permissions the agent is using

Not every agent should operate in the same way.

Some agents act on behalf of a signed-in user. The agent should then remain inside that user’s delegated permissions. If I cannot read a customer record, an agent acting for me should not be able to read it either.

Other agents run unattended. They act on their own schedule or in response to an event, without a human user in the loop. Those agents need their own application permissions and a much stricter definition of what they are allowed to do.

This is an architectural decision, not only an authentication setting.

For every agent, we should be able to answer:

1. Is it acting for a user or under its own authority?
2. Which identity is presented to each downstream system?
3. Which exact tools and operations can it use?
4. What data can it read, change, or send?
5. Who approved that access?
6. Who can disable the agent when something goes wrong?

If those answers are unclear, the agent is not ready for production.

## Ownership: Treat agents like employees, but stricter

Agents need the same discipline as employees: a known purpose, a responsible owner, appropriate access, regular reviews, and a clear end to the lifecycle.

When a new employee joins, we know who the manager is. We should know the same for an agent.

Microsoft Entra Agent ID has two roles for this. The **sponsor** is accountable for why the agent exists. The **owner** is the technical administrator.

It is similar to the split we already use for people: a manager who answers for the role, and IT who manages the account.

An agent with neither is an agent nobody will answer for when something goes wrong.

But their boundaries need to be tighter. Agents operate continuously, execute at machine speed, and can be influenced by prompts, retrieved data, tool descriptions, and responses from other systems. Unlike a human colleague, an agent does not stop because something feels wrong.

The old identity fundamentals still apply.

The tolerance for weak implementation should not.

## Controls: Seven practical places to start

We do not need to invent security again for AI agents.

We need to apply what we already know—more consistently and with much tighter boundaries.

I would start here:

1. **Inventory the actors.** Know which agents, MCP servers, tools, app registrations, managed identities, and federated identities exist.
2. **Remove static credentials.** Replace keys and client secrets with managed identities, agent identities, or workload identity federation wherever the target supports it.
3. **Separate production identities.** Give each production agent an identity that can be audited and disabled independently.
4. **Choose the authority model.** Be explicit about whether the agent acts on behalf of a user or under its own application permissions.
5. **Reduce the permission scope.** Assign the smallest suitable role at the narrowest practical resource scope. Avoid broad subscription-level access.
6. **Assign ownership and lifecycle.** Every agent needs a sponsor, technical owner, review date, and retirement path.
7. **Log the complete chain.** Record the caller, agent, tool, downstream identity, operation, and result so an investigation can reconstruct what happened.

This is not bureaucracy around AI.

It is what makes AI safe enough to become part of real business processes.

## Guardrails: Two kinds, and we need both

What about guardrails?

Guardrails are an important part of agent security, but the term covers different types of controls.

Model-level guardrails can filter harmful content, detect prompt attacks, and validate inputs and outputs. Microsoft Foundry's [guardrails and controls](https://learn.microsoft.com/en-us/azure/foundry/guardrails/guardrails-overview) work at this level.

Identity guardrails—such as least privilege and [Conditional Access for agents](https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id)—limit which resources an agent can access and under whose authority.

The important point is defense in depth:

> Behavioral guardrails reduce the likelihood of a bad decision. Identity and authorization limit the impact when one still happens.

We need both.

And the identity guardrails only work when the agent actually uses an identity. An agent that connects with an API key bypasses Conditional Access completely.

## Drift: The designed identity may not be the identity in use

An architecture diagram may show a managed identity. The deployed configuration may still contain a connection string. The configured identity may have broader permissions than intended, while the running solution may use another identity entirely.

And even when the original design was correct, access and ownership can drift over time.

This is the recurring governance gap we need to understand:

- What was designed
- What was configured
- What is running
- What is actually being used
- What has changed

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/governance-gap-mobile.svg">
  <img src="/assets/images/2026/governance-gap.svg" alt="The governance gap between architecture intent (designed, configured) and operational reality (running, actually used, changed), closed by the continuous loop discover, understand, prioritize, improve, verify">
</picture>

Identity governance cannot stop when the first role assignment is created. It needs continuous verification throughout the workload lifecycle.

## Governance: From findings to direction

This is also how we think about security in Helium, our service for continuous insight into Azure environments.

Today, Helium surfaces managed-identity signals, static-key findings, public exposure, networking, and resource configuration. That helps teams move from failed checks to context and priority.

The continuous-governance loop is:

> **Discover → Understand → Prioritize → Improve → Verify**

A static credential is a finding. But the useful part is understanding why it matters in this particular environment:

- Is an identity-based alternative available but not used?
- Is the resource also publicly exposed?
- Which step toward a more mature security posture does it block?
- What should the team fix first?

Security findings become useful when they create clarity and direction—not only another list of failed checks.

The next opportunity is to correlate identities and permissions with Azure resources, exposure, activity, ownership, and architectural context. Over time, that could include agent identity observability, effective-permission mapping, and a broader identity graph.

Helium does not replace Microsoft Entra, PIM, Defender, or Sentinel. Its role is to connect their controls and signals with the wider Azure environment so teams can understand what matters first and verify whether it improves.

## Accountability: So, who owns the identity?

The answer is the same as it was for the architecture:

> We still do.

The agent does not own the permissions, the data it exposes, the action it takes, or the incident it might create. The organization deploying it is still accountable.

The security fundamentals have not disappeared.

Identity. Least privilege. Zero Trust. Ownership. Visibility.

They have simply become more urgent.

## Boundaries: Identity is not the only perimeter

Identity tells us who or what is allowed to request access.

It does not answer where the agent can connect, which tools it can reach, or where data can leave.

That is the next layer: boundaries. [Azure Network Security Perimeter](https://learn.microsoft.com/en-us/azure/private-link/network-security-perimeter-concepts), gateways, tool access, and data-exfiltration paths all enter the picture.

That is where the next article, **Identity Is Not the Only Perimeter**, begins. After boundaries, we return to validation: what could the agent do, what did it do, and can we reconstruct the complete chain?

### If I need some assistance?

We at DevUP work with exactly this challenge: helping organizations understand identities, static credentials, exposure, and ownership across their Azure environments, using our service **Helium** for continuous insight and verification.

Reach out here:

- [https://www.devup.solutions/](https://www.devup.solutions/)
- [Email: mattias@devup.solutions](mailto:mattias@devup.solutions)
- [Read more about our perspective on governing AI-generated solutions](https://www.devup.solutions/usecases/secure-ai-generated-code)
