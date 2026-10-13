---
layout: post
title: "Who Decides What the Agent Can Access?"
description: "At an event last week, the room named security as the biggest problem with AI agents. With an agent in the loop, security is not one control. It is several layers, and each layer carries a decision someone needs to own and a constraint someone needs to verify."
date: 2026-10-14 09:00:00 +0200
categories: [Helium, AI]
tags: [AI, Security, Azure, Governance, Least Privilege, Data Access, Agent ID, Helium]
author: "Mattias Lögdberg"
image: /assets/images/2026/agent-access-hero.png
comments: true
---

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/agent-access-hero-mobile.svg">
  <img src="/assets/images/2026/agent-access-hero.svg" alt="An order agent next to a stack of access layers: identity, permissions, data scope, tools, instructions, path, approval, evidence. Each layer has a named owner, except permissions, which was widened at the keyboard and has no owner. Headline: Who decides what the agent can access?">
</picture>

Last week I ran a session at an event on the same question this series started with: [Who Owns the Architecture When AI Writes the Code?]({% post_url 2026-09-07-who-owns-the-architecture-when-ai-writes-the-code %})

For most of the session I used a whiteboard instead of slides. I used one business process as the example. A customer sends an order. An agent reads it, checks the prices, creates the order in the business system, and sends a confirmation.

When I asked the room where this gets difficult in their organizations, the answer was security.

Not cost. Not quality. Not that the model is wrong. Security.

After the discussion that followed, I do not think that answer is strange at all.

This article is not one of the four numbered articles in the series. It is a spin-off, in the same way as [Governance Needs to Move at the Same Speed as AI]({% post_url 2026-09-23-governance-needs-to-move-at-the-same-speed-as-ai %}), and it sits between two of them. [Every AI Agent Needs an Identity — and an Owner]({% post_url 2026-10-01-every-ai-agent-needs-an-identity %}) answered who is acting. The next numbered article, **Identity Is Not the Only Perimeter**, looks at where the agent can connect and what can pass through those connections.

This one is about the question that sits between those two articles, and that came up in the room. With an agent in the loop, where do the constraints live, and who decides what the agent is allowed to access?

<picture>
  <img src="/assets/images/2026/agent-access-whiteboard.webp" alt="Whiteboard at the end of the session. Left: the development flow with and without AI. Middle: the customer order process with an order agent, customer documents, pricing data, a question mark for the owning team, and a six-month timeline. Right: the Azure environment inside the loop: discover, understand, prioritize, improve, verify. Bottom: today's pipeline, developer to pull request to deploy, next to a future one: generated change, policy validation, dependency analysis, architecture analysis, compliance verification, runtime monitoring.">
</picture>

*The board at the end of the session. Most of this article is on it.*

## Layers: Security with an agent in the loop is not one control

For a classic integration, we are used to answering the security question in two or three places. The service has an identity. The identity has a role. The endpoint is private or it is not.

Put an agent in the same flow and the question spreads out.

The order agent needs the customer's documents. It needs the pricing data. It needs permission to create an order. It may need to approve a discount. And it decides, at runtime, which of those it uses and in which order.

That means the constraint can live in several places at once:

- **The identity** the agent runs as, and whether it acts for a user or under its own authority.
- **The permissions** on each resource: the role, and the scope the role is assigned at.
- **The data** inside the resource: which documents, which customers, which price list.
- **The tools** exposed to the agent, and what each tool can do. Read a price. Create an order. Change the pricing rules.
- **The instructions**: the system prompt and model guardrails that shape what the agent tries to do.
- **The path**: which networks and endpoints the agent can reach, and through which gateway.
- **The approval**: the point where a human decides, such as a discount above an agreed limit.
- **The evidence**: the logs that show what the agent actually did.

Every one of those layers answers a different question. None of them, on its own, is "security".

That is what makes the room's answer reasonable. When people say security is the biggest problem, I think they mean something more specific: we are not sure which of these layers we have, which ones we are missing, and whether they are configured the way we believe.

> **With an agent in the loop, security is a stack of decisions. The layer nobody owns is the one that drifts.**

## Decisions: Somebody decides at the keyboard

This was the part of the whiteboard that generated the most discussion.

Six months after the order agent went live, somebody needed more access. Something did not work. A permission was widened to get it working. Perhaps the agent got read access to the whole storage account instead of one container. Perhaps it got write access to the pricing data because a test needed it.

Nobody made a strange decision. They solved a problem.

But look at who decided what the agent can access: the person who was blocked, at the moment they were blocked, with the smallest change that made the error go away.

That is the default decision model in most environments today. The question "what should this agent be allowed to access?" is answered by whoever is unblocking the deployment.

Data ownership makes it worse. In the example, the customer documents and the pricing data belong to other teams. The team building the agent does not own them. The builder is now deciding, on behalf of the data owner, how much of that data an agent sees.

So who should decide? I think it is at least four different roles. The problem is that we rarely name them.

| Decision | Who should own it |
| --- | --- |
| Why the agent exists and which business process it serves | The business sponsor |
| Which data is in scope for that process, and how much of it | The owner of the data |
| The narrowest technical scope that still does the job | The team building the agent |
| The non-negotiables every agent must meet | The platform or security team |

None of those decisions should be made at the keyboard when a deployment fails.

A permission change that widens what an agent can reach is an architecture change. It needs the same owner and the same review as any other architecture change.

## Scope: Give the agent the document, not the storage account

The most common question after the session was practical. How do we make sure the data the agent accesses is limited to the targeted scope?

The principle is simple to state and harder to keep:

> **The agent should receive the data the task needs, not the credentials to go and find it.**

For the order agent, the targeted scope is the documents for this order, the current price for these items, and the permission to create one order. Not every customer's documents. Not the price history. Not the ability to change prices.

In Azure, that scope can be enforced at several levels, and the levels stack:

1. **Scope the role.** Azure RBAC assigns a role at a [scope](https://learn.microsoft.com/en-us/azure/role-based-access-control/scope-overview). Storage Blob Data Reader on one container is a very different thing from the same role on the subscription. The identity article asked for the narrowest practical scope. Here, I would add: treat every widening as a change.
2. **Separate the data by purpose.** If the order documents sit in the same container as the contracts, no role assignment will separate them. Data that different agents and processes need should not share a boundary just because it was convenient when it was created.
3. **Put the scope in the tool.** An agent calling a tool named *get price for item* cannot ask for the whole price list. A tool that runs an arbitrary query can.
4. **Keep the user's authority with the backend.** When the agent acts for a user, the system that owns the data should decide what that user may see. The next article covers this in detail.
5. **Filter retrieval, not only storage.** If documents are indexed for retrieval, the index needs the same scope as the source. An index that contains everything, queried by an agent that can ask anything, has quietly removed the boundary we built around the storage account.
6. **Pass the data in where you can.** For a well-defined process, the orchestration can hand the agent the three documents for this order. The agent then has no search permission to misuse.

Each of those is a decision. Each has an owner from the table above. And each can drift.

## Constraints: You cannot verify what you never agreed

The other discussion that stayed with me was about where the constraints live.

On the board, we ended with an agreed target next to the environment. For the order agent, it looked something like this:

- Sensitive services use private access.
- Managed identity or agent identity where the service supports it. No shared keys.
- Each production agent has its own identity, a sponsor, and an owner, as in the identity article.
- Data access is scoped to the process, and the scope is written down.
- Write operations are named. Create order: yes. Change pricing: no.
- Discounts above an agreed limit require a human approval.

None of that is exotic. The point is that it is written down, so the current state can be compared with it.

When we then check the environment and find that the agent's identity has read access to the whole storage account, we have a gap, not an opinion. We can discuss whether there is a reason for it, whether we accept an exception with an owner and an expiry, or whether it needs to change. And we can measure whether the gap is getting smaller.

> **A constraint nobody wrote down cannot be verified. It can only be found after an incident.**

## Pipeline: Review moves from reading the code to checking the constraints

The last thing we drew was the delivery pipeline, twice.

Today: developer, pull request, human review, deploy. A person reads the change and decides whether it looks right.

That worked when humans wrote code at human speed. In the first article of this series, I argued that review stopped scaling. When the implementation is generated, reading the diff tells us less and less about whether the change stays inside what we agreed.

What we sketched instead was a simpler version of the pipeline from that article, where a generated change passes through checks against the constraints:

> **Generated change → Policy validation → Dependency analysis → Architecture analysis → Compliance verification → Runtime monitoring**

- **Policy validation.** Does the change meet the non-negotiables? [Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/overview) can deny what is never acceptable before it exists.
- **Dependency analysis.** What does this change connect to? Which data, which identities, which other team's resources?
- **Architecture analysis.** Is the new access inside the agreed scope for this process, or did it just get wider?
- **Compliance verification.** After deployment, is the environment still inside the target?
- **Runtime monitoring.** Is the agent using the access we think it is, and nothing else?

That is a drawing, not a product. Parts of it exist today in Azure Policy, Azure Resource Graph, Defender for Cloud, and the logging the identity article asked for. The point is that the question asked in the review changes:

> **Not "does this code look right?" but "does this change stay inside what we agreed the agent may access?"**

And the question keeps being asked after deployment. The permission that was widened six months later never went through a pull request.

That is the same loop as in the governance article:

> **Discover → Understand → Prioritize → Improve → Verify**

## Governance: Where Helium fits

This is also how we approach it in Helium, our Continuous Cloud Governance Platform for Azure.

Helium does not decide what an agent should access. That stays with the owners in the table above. It does not see inside the agent's prompt, and it does not know which three documents a task needs.

What it does is compare the Azure environment with the requirements the organization has agreed: which identities exist, which resources still use keys where an identity is available, what is publicly exposed, and what has changed. With context, a static key on an agent's resource becomes a priority rather than one more finding.

That gives the people who own the decisions something concrete to decide on. And it shows whether the gap is closing after they do.

## Accountability: So, who decides?

We still do.

Not the agent. Not the model. Not the person who happened to be unblocking a failed deployment.

The room was right that security is the biggest problem. I would phrase it slightly differently. Security has become a stack of layered decisions, and most organizations have not yet decided who owns each layer, or written down the constraint they want to verify.

Identity tells us who is acting. Scope tells us what it may touch. The next article follows one of those layers further: where the agent can connect, which tools it can reach, and what can pass through an approved connection.

Identity is not the only perimeter.

---

**Want to compare your Azure environment with what you have agreed it should look like?**

DevUP Helium is our Continuous Cloud Governance Platform for Azure. [Learn more about Helium](https://www.devup.solutions/) or [contact Mattias](mailto:mattias@devup.solutions).
