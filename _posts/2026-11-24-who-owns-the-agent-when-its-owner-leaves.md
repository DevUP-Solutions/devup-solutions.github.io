---
layout: post
title: "Who Owns the Agent When Its Owner Leaves?"
description: "Every agent needs a sponsor and an owner. People change roles and leave. The tooling to hand an agent over now exists. The decision about who inherits it is still ours, and it has to be made before the person leaves."
date: 2026-11-24 09:00:00 +0100
categories: [Helium, AI]
tags: [AI, Security, Azure, Microsoft Entra, Agent ID, Lifecycle Workflows, Governance]
author: "Mattias Lögdberg"
comments: true
---

In [Every AI Agent Needs an Identity — and an Owner]({% post_url 2026-10-01-every-ai-agent-needs-an-identity %}), I said every agent needs "a sponsor, technical owner, review date, and retirement path."

I wrote a lot about the identity. I wrote very little about what happens to the sponsor over time.

People change roles. People leave. The agent does not notice.

This post is about that gap. It is a spin-off before the final article in the series, the way [Who Decides What the Agent Can Access?]({% post_url 2026-10-21-who-decides-what-the-agent-can-access %}) was before [Identity Is Not the Only Perimeter]({% post_url 2026-11-10-identity-is-not-the-only-perimeter %}).

## Precedent: We have seen this with applications

Most Azure tenants I look at have an application nobody owns anymore.

The person who registered it left. The secret still works. Something still calls it, or might. Deleting it feels riskier than leaving it, so it stays.

TODO(Mattias): the orphaned app registration you see most often in customer tenants. The shape is enough: who created it, what it still does, why nobody dares to remove it. No customer name.

Agents will reach the same state faster. They are cheaper to create, often created outside the platform team, and they keep working when nobody is watching.

An application with no owner is a finding. An agent with no owner is a finding that can still make decisions.

## Roles: Two names, one gap

Microsoft Entra Agent ID has two roles for an agent. The [sponsor](https://learn.microsoft.com/en-us/entra/agent-id/agent-owners-sponsors-managers) is accountable for why the agent exists. The owner administers it.

The documentation is clear that "At least one sponsor is required for each agent identity and agent identity blueprint." Owners are optional.

The same page also says "Sponsorship should be maintained to ensure succession when an employee who's a sponsor moves or leaves."

Should be maintained. By whom?

The required role is the one that is easiest to forget. Nobody has to administer the agent for it to keep running. The sponsor's job is to answer for it. When the sponsor leaves, nothing breaks, so nothing tells us.

## Mechanism: The hand-over now exists

Microsoft has shipped the plumbing for this. Entra [lifecycle workflows](https://learn.microsoft.com/en-us/entra/id-governance/agent-sponsor-tasks) now have three sponsor tasks: email the manager about sponsorship changes, email the cosponsors, and transfer the agent identity sponsorships to the manager.

They run inside the mover and leaver workflows we already use for people. There are two [templates](https://learn.microsoft.com/en-us/entra/id-governance/lifecycle-workflow-templates): one for when a sponsor leaves, one for when a sponsor changes roles. Microsoft describes the purpose plainly in [what's new for Agent ID](https://learn.microsoft.com/en-us/entra/agent-id/whats-new-agent-id): "to prevent orphaned agents."

At Ignite last week, Microsoft also announced an [action center in Agent 365](https://techcommunity.microsoft.com/blog/agent-365-blog/whats-new-in-microsoft-agent-365-ignite-2026/4571753) that starts by "surfacing inactive and ownerless agents for review and cleanup." It is announced for preview in December. The same post puts the problem in one line: "An agent without an owner has no one accountable for it, and an inactive agent can keep access it no longer needs."

Three things are worth knowing before relying on any of it.

The transfer task needs "a populated manager attribute for the user." If the manager field is empty or wrong, there is nobody to transfer to.

The sponsor templates require an Agent 365 license.

And none of this covers the agents that never got an Agent ID. The agent that runs on a shared app registration, or a key in a Logic App, has no sponsor to transfer.

## Decision: The workflow is not the policy

A workflow can move a sponsorship. It cannot decide whether the new sponsor should accept it.

The default transfer goes to the sponsor's manager. That is a reasonable default. It is also a person who may not know the agent exists, what it reads, or which process depends on it.

So the questions are ours to answer, and they need answers before someone resigns:

- **Who inherits by default?** The manager, a named team, or a group sponsor. Entra allows groups as sponsors. A team outlives a person.
- **What does the new sponsor have to confirm?** That the agent is still needed, that its access still matches what was agreed, and that its review date still makes sense.
- **What happens if nobody confirms?** Microsoft's [best practices](https://learn.microsoft.com/en-us/entra/agent-id/best-practices-agent-id) say: "If a sponsor doesn't confirm, evaluate the agent for decommissioning." I would go one step further and disable it, not delete it. A disabled agent can be restored. A deleted one cannot answer questions in an investigation.
- **How often do we look?** The same page recommends a quarterly review for "agents with missing sponsors, outdated metadata, or no recent activity," and sponsor attestation every six to twelve months.

The [access post]({% post_url 2026-10-21-who-decides-what-the-agent-can-access %}) asked who decides what an agent may access. This is the same question, asked again every time the person who decided leaves.

## Experience: The small-company version

TODO(Mattias): the agent framework. Every agent session runs under a scheduled task that one person owns: you. If you were away for a month, which of those sessions should stop, which should keep running, and who would know the difference? One honest paragraph.

TODO(Mattias): Helium's own Entra registrations, such as the MCP server app. Who is listed as owner today, and what would happen if that person left DevUP? A one-line honest answer is stronger than a hypothetical.

## Start: With the inventory you already have

You do not need Agent 365 to start.

1. **List the applications with no owner.** App registrations and service principals with zero owners, or owners no longer in the tenant. Fix those first. They are the precedent.
2. **Check the manager attribute.** For everyone who sponsors or owns an agent or application. The transfer depends on it.
3. **Prefer group sponsors for production agents.** A team, not a person.
4. **Add the sponsor templates to your leaver and mover workflows,** where you have the license. Where you do not, add a manual step to the offboarding checklist.
5. **Disable on silence.** An agent whose sponsor does not confirm within an agreed time is disabled, not left running.

The [governance post]({% post_url 2026-09-23-governance-needs-to-move-at-the-same-speed-as-ai %}) said "A resource without an owner should be a finding on day one." The same applies to agents. The difference is that the day an agent loses its owner is not the day it was created. It is the day someone hands in their notice.

## Accountability: So, who owns it now?

> We still do.

The organization owned the agent when the sponsor was here. It still owns it after they leave. The question is only whether anyone in that organization knows.

The next and final article in the series asks whether we can reconstruct what an agent did. That gets much harder when the person who could have told us has already left.

---

**Want to see which resources and identities in your Azure environment have no clear owner?**

[Learn more about DevUP Helium](https://www.devup.solutions/) or [contact Mattias](mailto:mattias@devup.solutions).
