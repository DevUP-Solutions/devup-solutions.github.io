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
  <img src="/assets/images/2026/governance-speed-hero.svg" alt="AI-assisted delivery rapidly adding new resources to an Azure environment of applications, identities, data, networking, and integrations, with a continuous governance loop following every change underneath. Headline: Delivery is continuous. Governance must be too.">
</picture>

In my previous article, [Who Owns the Architecture When AI Writes the Code?]({% post_url 2026-09-07-who-owns-the-architecture-when-ai-writes-the-code %}), I argued that AI has changed implementation, but it has not transferred responsibility.

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

Those checkpoints still have value, but they only show the environment at a specific moment.

Azure does not stop changing after the review.

A team adds a resource. Someone creates an exception. A permission is widened. A public endpoint is enabled for troubleshooting.

None of those changes looks dramatic on its own. Together, they change the security and architecture of the environment. When delivery becomes continuous but governance remains periodic, drift becomes the normal state.

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/periodic-vs-continuous-governance-mobile.svg">
  <img src="/assets/images/2026/periodic-vs-continuous-governance.svg" alt="Two timelines of the same environment. Periodic governance: architecture review, deployment approval, and annual assessment as isolated checkpoints, with unmanaged changes accumulating between them, ending in a long remediation backlog and another review later. Continuous governance: the same checkpoints remain, but every change is picked up by the loop discover, understand, prioritize, improve, verify, and repeat.">
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

## Guardrails help, but they are not the complete model

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

## How can we solve this?

The good news is that many of the building blocks already exist in Azure.

[Azure Resource Graph](https://learn.microsoft.com/en-us/azure/governance/resource-graph/overview) can help us understand what is deployed across subscriptions. Azure Policy can audit and enforce requirements. Defender for Cloud, Azure Advisor, Azure Monitor, and Cost Management provide additional signals across security, reliability, operations, and cost.

We can connect those services through scripts, workbooks, automation, and backlogs. We can assign owners, document exceptions, prioritize findings, and regularly check whether the environment has improved.

That can work.

The challenge is bringing it all together.

Individual services provide valuable controls and signals, but they do not automatically create a shared view of risk, context, dependencies, ownership, priorities, and progress. Someone still needs to connect the information, remove the noise, maintain the rules, and keep the process moving.

As the number of subscriptions, services, teams, and requirements grows, maintaining the governance system can become a product of its own.

That is where Helium fits.

DevUP Helium provides continuous cloud governance for Azure. It brings signals, context, priorities, improvements, and verification into the same governance loop.

Helium continuously evaluates the Azure environment across security, compliance, operational excellence, architecture, reliability, and cost.

It surfaces issues such as static credentials where managed identities are available, public exposure, missing diagnostics, unsupported versions, weak TLS, and missing operational controls.

But the number of failed checks is not the point. The point is to understand what the findings mean together, what to improve first, and whether the environment is becoming better.

That is also why AI matters to the Helium story. AI increases the rate of change. It can also help us understand relationships and produce clearer recommendations. Both make continuous visibility and verification more important.

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

### If I need some assistance?

We at DevUP work with exactly this challenge: helping organizations keep governance up to speed with AI-assisted delivery, using our service **Helium** for continuous cloud governance across Azure environments.

Reach out here:

- [https://www.devup.solutions/](https://www.devup.solutions/)
- [Email: mattias@devup.solutions](mailto:mattias@devup.solutions)
- [Read more about our perspective on governing AI-generated solutions](https://www.devup.solutions/usecases/secure-ai-generated-code)
