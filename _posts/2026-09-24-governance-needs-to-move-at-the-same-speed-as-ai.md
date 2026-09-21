---
layout: post
title: "Governance Needs to Move at the Same Speed as AI"
description: "AI does not change what we need to govern in Azure. It changes how quickly the environment grows and drifts. When delivery is continuous, governance has to be continuous too: discover, understand, prioritize, improve, verify."
date: 2026-09-24 09:00:00 +0200
categories: [Helium, AI]
tags: [AI, Governance, Azure, Azure Governance, Continuous Governance, Architecture, Helium]
author: "Mattias Lögdberg"
image: /assets/images/2026/governance-speed-hero.png
comments: true
---

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/governance-speed-hero-mobile.svg">
  <img src="/assets/images/2026/governance-speed-hero.svg" alt="AI-assisted delivery rapidly adding new resources to an Azure environment of applications, identities, data, networking, and integrations, with a continuous governance loop following the environment underneath. Headline: Delivery is continuous. Governance must be too.">
</picture>

In my previous article, [Who Owns the Architecture When AI Writes the Code?]({% post_url 2026-09-07-who-owns-the-architecture-when-ai-writes-the-code %}), I argued that AI has changed implementation but has not transferred responsibility.

We still own the architecture.

One part of that discussion deserves a closer look:

> **Governance needs to move at the same speed as AI.**

That does not mean governing AI in isolation. We are still responsible for the Azure environment: its resources, identities, dependencies, exposure, security, reliability, cost, and lifecycle.

AI changes how quickly that environment grows and changes. That is what makes the governance question urgent.

## AI is the accelerant, not the governance target

Most of the Azure problems we see today are not new.

We already had public endpoints that should have been private, static credentials where managed identities were available, overly broad permissions, missing diagnostics, unsupported runtimes, and resources without a clear owner.

AI did not invent any of these problems.

What AI changes is how cheaply and quickly we can reproduce them.

A developer can now ask for an Azure Function, Storage account, managed identity, Service Bus queue, infrastructure template, and deployment pipeline—and get a convincing first implementation in minutes.

That is useful. I use AI in development myself, and I do not think the answer is to slow it down.

Every generated resource still becomes part of an environment someone needs to understand, secure, operate, pay for, and eventually retire.

The challenge is not only whether the generated code is correct. It is whether the Azure environment remains intentional.

## Traditional governance sees moments in time

Many governance models are built around checkpoints. We review the architecture, approve the pull request, validate the deployment pipeline, and perhaps revisit the architecture a few times per year.

Those checkpoints still have value, but they are snapshots of the environment at a specific moment.

Azure does not stop changing after the review.

A team adds a resource. Someone creates an exception. A permission is widened. A public endpoint is enabled for troubleshooting.

None of those changes looks dramatic on its own. Together, they change the security and architecture of the environment. When delivery becomes continuous but governance remains periodic, drift becomes the normal state.

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/periodic-vs-continuous-governance-mobile.svg">
  <img src="/assets/images/2026/periodic-vs-continuous-governance.svg" alt="Two timelines of the same environment. Periodic governance: architecture review, deployment approval, and annual assessment as isolated checkpoints, with unmanaged changes accumulating between them, ending in a long remediation backlog and another review later. Continuous governance: the same checkpoints remain, but the changes between them are picked up by the loop discover, understand, prioritize, improve, verify, and repeat.">
</picture>

*Periodic governance captures moments. Continuous governance follows the environment.*

By the next review, we may no longer be validating the architecture we designed. We are reconstructing what happened since the last time we looked.

Governance that arrives after the environment has changed is mostly documenting what already happened.

## A green deployment is not a governed environment

A deployment pipeline can tell us that the template was valid and the deployment succeeded. It may also run tests, security scans, and policy checks.

That matters.

But a green deployment only tells us that something deployed successfully. It does not tell us whether it should exist, whether it is still secure six months later, or whether anyone understands what depends on it.

It does not answer questions such as:

- Is the assigned identity broader than the workload requires?
- Did the resource become publicly accessible?
- Which systems depend on it?
- Who owns it six months from now?
- Is the complete environment becoming better or only larger?

The pipeline sees the change it is deploying. Governance needs to understand the environment receiving that change.

That difference becomes increasingly important when AI is good at producing locally reasonable solutions without understanding the complete system around them.

## Guardrails help, but they are not governance

Guardrails are an important part of the answer.

We can use Azure Policy, secure templates, least-privilege identities, deployment scopes, network boundaries, approved resource types, and human approval for sensitive changes.

At the agent level, Microsoft Foundry guardrails can inspect input, tool calls, responses, and output. Some capabilities remain in preview, but controls are clearly moving closer to the actions an agent takes. [Microsoft describes the current model here](https://learn.microsoft.com/en-us/azure/foundry/guardrails/guardrails-overview).

In a recent [DevUP Talks conversation with Simon Wåhlin](https://youtu.be/-eAswM6jzxA), we discussed an example where AI tried to solve a connectivity problem by removing the firewall that blocked it.

It solved the immediate problem—and removed the security boundary.

A guardrail could prevent that action. Azure Policy can deny, audit, modify, or remediate resources that do not meet organizational requirements. [Read the Azure Policy overview](https://learn.microsoft.com/en-us/azure/governance/policy/overview).

But preventing one bad change is not the same as governing the environment.

Guardrails define what should be allowed, blocked, inspected, or escalated. Governance also needs to set the direction, handle justified exceptions, understand the context, prioritize improvement, and verify that the controls still work.

> **Guardrails provide boundaries. Governance provides direction, responsibility, and follow-through.**

## Governance needs to become continuous

Continuous governance does not mean inspecting every setting in real time or blocking every deviation. It means that governance follows the environment throughout its lifecycle instead of appearing only at selected checkpoints.

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/continuous-governance-loop-mobile.svg">
  <img src="/assets/images/2026/continuous-governance-loop.svg" alt="The continuous governance loop. Discover: what is actually deployed? Understand: what does it mean in context? Prioritize: what matters first? Improve: what action should be taken? Verify: did the environment become better? And repeat.">
</picture>

In practice, I think it comes down to five things.

### 1. Discover the actual environment

Start with what is really deployed—not only what appears in the architecture diagram or repository. Which resources exist? Which identities do they use? What is exposed and connected?

Without an accurate view of the current state, every later governance decision starts from an assumption.

### 2. Understand the context

A failed check does not automatically explain the risk.

A public endpoint on an intentionally public API is different from one on an internal data store. A static key becomes more urgent when an identity-based alternative exists. A missing alert matters more on a critical production flow than on a temporary development resource.

Governance needs context: purpose, exposure, dependencies, activity, ownership, environment, and business impact. That is what turns configuration data into a decision.

### 3. Prioritize what matters

Most Azure environments contain more findings than a team can fix at once. Another list of everything that is technically wrong does not solve that problem.

Teams need to know:

- What creates the largest risk?
- Which improvement reduces several risks at the same time?
- What should we address in the next sprint?

Governance becomes useful when it creates direction—not when it creates more noise.

### 4. Improve deliberately

Some controls should prevent deployment. Others should create a finding or a prioritized backlog item. There will also be exceptions.

The goal is not to pretend that every environment can become perfect overnight. The goal is to make the trade-offs visible, give exceptions an owner and an expiry, and improve the environment step by step.

### 5. Verify the result

After the team acts, governance needs to check again. Was the public endpoint removed? Is the managed identity now used? Did the security posture improve?

> **Discover → Understand → Prioritize → Improve → Verify**

Without verification, remediation becomes another moment in time.

## Continuous governance is not continuous control

Continuous governance can sound like putting more controls around developers. That is not the goal. The goal is to spend less time investigating what we have and more time improving it.

Some decisions can be automated:

- Deny a configuration that is never acceptable
- Detect drift
- Identify an unsupported version
- Show when identity-based authentication is available but not used

People still need to decide whether an exception is justified, which risks can be accepted temporarily, and what should be improved first. Automation should make that possible at a scale where manual review is no longer realistic—not add another approval board.

AI can help here too. The same technology that increases the rate of change can also help us understand relationships and produce clearer recommendations.

## How can we solve this?

Many of the building blocks already exist in Azure.

[Azure Resource Graph](https://learn.microsoft.com/en-us/azure/governance/resource-graph/overview) can help us understand what is deployed across subscriptions. Azure Policy can audit and enforce requirements. Defender for Cloud, Azure Advisor, Azure Monitor, and Cost Management provide additional signals across security, reliability, operations, and cost.

We do not need a new product to get started. To put the continuous governance loop into practice, I would start here:

1. **Build one inventory.** Query what is actually deployed across all subscriptions: resources, identities, and public exposure. Start from the environment—not from the diagram.
2. **Make ownership a deployment requirement.** Use Azure Policy to require tags for owner, environment, and criticality. A resource without an owner should be a finding on day one, not in next year's assessment.
3. **Choose a few non-negotiables.** Deny the configurations that are never acceptable. Audit the rest. Trying to block everything mostly creates exceptions nobody follows up.
4. **Give every exception an owner and an expiry.** Azure Policy exemptions support expiration dates. Use them. An exception without an end date quietly becomes part of the architecture.
5. **Bring the signals into one prioritized list.** Policy compliance, Defender for Cloud, Advisor, Monitor, and Cost Management each tell part of the story. Rank the findings by context, not by count, and bring the top few into the next sprint.
6. **Check again after every fix.** Follow whether the environment is becoming better over time—not only how many findings are open.
7. **Match the cadence to delivery.** If the environment changes every day, a yearly review can only document what already happened.

Scripts, workbooks, automation, and a backlog will take us a long way.

That can work.

The challenge is keeping it all together.

Individual services provide valuable controls and signals, but they do not automatically create a shared view of risk, context, dependencies, ownership, priorities, and progress. Someone still needs to connect the information, remove the noise, maintain the rules, and keep the process moving.

As the number of subscriptions, services, teams, and requirements grows, maintaining the governance system can become a product of its own.

That is where Helium fits.

DevUP Helium provides continuous cloud governance for Azure across security, compliance, operational excellence, architecture, reliability, and cost.

Two parts of the governance loop are difficult to build and maintain yourself.

The first is the knowledge inside the checks. Azure does not stand still: new services, new authentication options, retired runtimes, and older TLS versions. Every check needs to be written, kept up to date, and explained well enough for a team to act on it.

The second is context.

A static key is a finding. A static key on a publicly exposed resource, where a managed identity is available but not used, is a priority.

The number of failed checks is not the point. The point is to understand what the findings mean together, what to improve first, and whether the environment is becoming better.

Helium does not replace Azure Policy, Defender for Cloud, or Advisor. They remain the building blocks. Helium's role is to keep the loop moving, so the team can spend its time improving the environment instead of maintaining the governance system.

The governance target remains the Azure environment.

## Governance has to follow what we deploy

AI will continue to make implementation faster. That is an opportunity.

But if delivery accelerates while governance remains periodic, the gap between what we intended to build and what we are actually operating will continue to grow.

We will not solve that with more documents, more meetings, or by trying to review generated output line by line at machine speed.

We solve it by making governance part of how the Azure environment is built, operated, and improved:

> **Discover the actual state. Understand the context. Prioritize the risk. Improve deliberately. Verify the result.**

AI may write the code.

Azure still runs what we deploy.

And governance needs to keep up with both.

The next article will look more closely at one part of that environment that AI is making increasingly important: identity. Because every AI agent needs an identity—and an owner.

---

**Want to make governance continuous?**

DevUP Helium is our Continuous Cloud Governance Platform for Azure. [Learn more about Helium](https://www.devup.solutions/) or [contact Mattias](mailto:mattias@devup.solutions).
