---
layout: post
title: "Can You Reconstruct What the Agent Did?"
description: "Identity decides who may act. Boundaries limit where. Neither tells you what actually happened. An agent incident you cannot reconstruct is an incident you cannot own."
date: 2026-12-08 09:00:00 +0100
categories: [Helium, AI]
tags: [AI, Security, Azure, Observability, OpenTelemetry, Microsoft Foundry, Microsoft Entra, Governance]
author: "Mattias Lögdberg"
comments: true
---

TODO(Mattias): hero image and social card (`image:` front matter), in the style of the identity and boundaries posts.

This is the last of four articles in this series.

In [Every AI Agent Needs an Identity — and an Owner]({% post_url 2026-10-01-every-ai-agent-needs-an-identity %}), I argued that every agent needs its own identity and a named owner. In Identity Is Not the Only Perimeter, I followed the paths an agent can use and the boundaries around them.

TODO(Mattias): replace the plain title above with a `post_url` link to `2026-11-10-identity-is-not-the-only-perimeter` once PR #19 is merged. Linking it now would break this branch's build.

Identity gives us the context. Boundaries define where the agent can act. Validation tells us what actually happened.

This one is about validation.

Both earlier articles ended with the same question. Can we reconstruct the complete chain?

## Incident: The first question is always the same

After an incident, someone asks what happened.

For a normal integration, we usually have an answer. A request came in through API Management. A Logic App ran. A message landed on a queue. We follow the correlation ID and read the story.

For an agent, the honest answer is often: we don't know.

We know a user asked something. We may know which model answered. But which tools did the agent call, with which arguments, under whose authority, and what changed in the target system?

That is the chain we need. Without it, we cannot say what the agent did. We can only say what it was allowed to do.

TODO(Mattias): a short first-hand example. An integration incident where a missing correlation ID made the investigation slow fits here.

## Evidence: The platform records more than it used to

The good news is that the building blocks exist.

[Foundry tracing](https://learn.microsoft.com/en-us/azure/foundry/observability/concepts/trace-agent-concept) is generally available for prompt and hosted agents. It records user inputs, agent outputs, tool calls with their results, token use and latency. It uses the OpenTelemetry semantic conventions for generative AI, with spans such as `invoke_agent` and `execute_tool`, and attributes for tool call arguments and results.

Boundaries can produce evidence too. When a hosted agent has [network egress controls](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/add-hosted-agent-guardrails), each outbound decision shows up as a "Network egress decision" span in the trace and in Application Insights. Audit mode logs what would have been denied.

And [Entra sign-in and audit logs](https://learn.microsoft.com/en-us/entra/agent-id/sign-in-audit-logs-agents) now mark agent activity. An `agentType` property tells you whether the actor was an agent identity, a blueprint, or an agent's user account.

That is real progress.

## Gaps: The chain is split across stores

The bad news is that each record sits in a different place.

The agent trace is in the project's Application Insights. The sign-in is in Entra. Tool traffic through an [AI gateway](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/governance) is in API Management logs, and the gateway itself does not log tool traces. Microsoft's own guidance is to use your MCP server's logs for tool-level detail.

So the chain crosses at least four log stores. And the most important link is the one nobody records by default: what the downstream system actually did with the request.

> A trace shows what the agent asked for. It does not show what the system did.

This is the integration problem again. We solved it for APIs and messaging with correlation IDs that travel with the request. Agents need the same discipline. The difference is that the agent decides at runtime which systems the request travels through.

In our own MCP server, the request to the backend API carries the correlation ID when the incoming request has one. That only helps if the caller sends one.

TODO(Mattias): what we see in practice. Do the MCP clients connecting to Helium send a correlation ID? If not, how do we tie an assistant's question to the backend calls today?

## Sensitivity: Evidence is also data

There is a catch.

To reconstruct the chain, we need the tool arguments. In the invoice scenario from the previous article, the tool arguments were the data that leaked.

Microsoft is direct about this. [Foundry trace data](https://learn.microsoft.com/en-us/azure/foundry/observability/concepts/trace-data) can contain prompts, responses and personal data. Anyone with read access to the connected Application Insights resource may see it. Tracing is off by default, and retention follows your Log Analytics settings.

So the evidence needs governance of its own:

- Who can read traces, and is that list shorter than the list of people who can read the source data?
- Are secrets and personal data redacted before they reach telemetry?
- How long do we keep the evidence, and does that match how long it takes to discover an incident?

An audit trail that leaks is not a control. It is another path.

## Proof: Evidence has to be designed before it is needed

We learned a version of this building our own agent framework.

When an agent changes what a background service writes to customer-visible storage, unit tests are not enough to close the work. The change needs a named live observation after the next real run: this document, in this container, with this shape. Until someone has seen it, the work is not done.

The same applies to agents in production. "The agent completed the task" is a claim. The evidence is what the target system recorded.

It also has to be tested. The time to discover that a log store is missing the one field you need is not during the incident.

## Start: Five places to begin

I would start here:

1. **Pick one correlation ID.** Carry it from the user's request through the agent, every tool call and every downstream write. Make tools and MCP servers pass it on, not just accept it.
2. **Turn tracing on deliberately.** Connect Application Insights, decide what content is recorded, and redact before it reaches telemetry.
3. **Log the decision points.** Egress decisions, gateway policy results and approvals are evidence that a boundary worked.
4. **Join the stores.** Know how to get from a trace to the Entra sign-in to the gateway log to the target system's own audit log. Write the query before you need it.
5. **Rehearse it.** Pick a normal agent run and reconstruct it end to end. Every step you cannot follow is a gap you found for free.

## Governance: Verify is the last step for a reason

This is the last step in the loop I have used through this series:

> **Discover → Understand → Prioritize → Improve → Verify**

Verify is where governance stops being an intention.

For an Azure environment, it means checking that the public endpoint is actually gone. For an agent, it means being able to show what it did, under whose authority, and what changed as a result.

## Accountability: So, who owns what the agent did?

We still do.

That has been the answer in every article. The architecture, the identity, the boundary. Now the outcome.

Owning it is easy to say. Proving it requires evidence.

The agent will not remember what it did. The platform will record pieces of it. Putting the pieces together is our job, and it is much easier to design before the incident than to reconstruct after it.

---

**Want to verify what is actually running in your Azure environment?**

DevUP Helium is our Continuous Cloud Governance Platform for Azure. [Learn more about Helium](https://www.devup.solutions/) or [contact Mattias](mailto:mattias@devup.solutions).
