---
layout: post
title: "Who Owns the Architecture When AI Writes the Code?"
description: "AI can accelerate implementation dramatically, but it does not take responsibility for the architecture, the production incident, or the customer impact. How do we keep architectural understanding when delivery starts moving at machine speed?"
date: 2026-09-07 09:00:00 +0200
categories: [Helium, AI]
tags: [AI, Architecture, Governance, Azure, Integration]
author: "Mattias Lögdberg"
comments: true
---

During the last few months, I have presented different versions of this topic at AVIATORS Germany in Hamburg, Integration Down Under, and Integrate.

The question grew from discussions we have been having at DevUP about what happens when delivery accelerates faster than governance. We want the speed and opportunity AI gives us, but we also need to understand what that speed does to the systems we are responsible for.

The title usually creates a good discussion:

**Who owns the architecture when AI writes the code?**

At first, this sounds like a question about AI tools. But I think it is really a question about responsibility, understanding, and how we build software when implementation is no longer the main constraint.

My short answer is simple:

> **We still own it.**

The longer answer is more interesting.

## AI is changing the speed of delivery

Historically, implementation was expensive.

A developer had to write the code, configuration, infrastructure, tests, mappings, policies, and deployment scripts. Human writing speed naturally limited how quickly a system could grow.

That limit is now changing.

AI can help us generate an API, a Function, a Logic App workflow, an infrastructure template, or a test suite in minutes. It can also repeat the same pattern across many services with very little additional effort.

This is a positive change. I use AI in development myself, and I do not think the answer is to slow it down.

But the acceleration is real. In its 2025 Octoverse report, GitHub reported almost one billion commits during the year, an increase of 25% year over year. Pull requests and code pushes also grew strongly, while comments on issues and pull requests were almost flat and comments on commits fell by 27%. GitHub is careful to describe these as observational signals rather than proof that AI caused the change, but the shape of it is interesting: production is accelerating faster than the coordination around it. [Read the GitHub Octoverse 2025 report](https://github.blog/news-insights/octoverse/octoverse-a-new-developer-joins-github-every-second-as-ai-leads-typescript-to-1/).

The 2025 Stack Overflow Developer Survey tells a similar story from another direction. It found that 84% of respondents were already using or planning to use AI tools in development, and 51% of professional developers used them daily. At the same time, more developers distrusted the accuracy of AI output than trusted it: 46% compared with 33%. [Explore the Stack Overflow 2025 AI survey](https://survey.stackoverflow.co/2025/ai).

So we are using AI more, producing more, and still not fully trusting what it produces.

That gap is where I think things start getting dangerous.

## The cost of producing mistakes has collapsed

AI does not necessarily create completely new categories of software problems.

We already had:

- Public endpoints that should have been private
- Identities with too many permissions
- Missing monitoring and diagnostics
- Undocumented dependencies
- Copied insecure patterns
- Abandoned resources
- Compliance drift
- Retry policies that turn a small failure into a storm

These are familiar problems.

What AI changes is the scale and velocity.

Before AI, significant human effort might result in one deployment and one mistake. Today, one prompt can create many related artifacts and repeat the same mistake across all of them.

The cost of producing software has dropped. Unfortunately, the cost of producing mistakes has dropped with it.

And a generated implementation can look very convincing. It compiles. The tests pass. The individual resource works.

That does not mean the system is correct.

## Integration teams will feel this early

This problem becomes very visible in integration and distributed cloud solutions.

In the Azure environments we work with at DevUP, the individual resource is rarely the difficult part. The challenge is understanding the complete environment: how everything is connected, why it was built, who owns it, and whether it still follows the intended architecture.

AI is often good at generating the individual parts independently:

- An API Management policy
- A Logic App
- An Azure Function
- A Service Bus topic
- A deployment pipeline
- An infrastructure-as-code template
- A monitoring query

But distributed systems rarely fail only because one component is obviously broken. They fail because of the relationships between components: a hidden dependency, an ownership gap, an incompatible retry strategy, a duplicated flow, or a change that makes sense locally but creates a problem globally.

Imagine a flow that starts in SAP, passes through API Management, a Logic App, Service Bus, a Function, and Cosmos DB before notifying three downstream systems.

Now ask:

- Who owns the complete flow?
- Who understands all its dependencies?
- Who notices when AI generates a second implementation beside the first one?
- Who knows which part is still needed six months later?

Every resource can be healthy while the architecture is slowly losing its intentionality.

## The Winchester Mystery House problem

Drew Breunig recently used the [Winchester Mystery House as a metaphor for what happens when code becomes cheap](https://www.dbreunig.com/2026/03/26/winchester-mystery-house.html).

The interesting part of the analogy is not simply that the house has strange architecture. Many of its individual additions were useful and made sense when they were built.

The problem appeared over time.

A room was added. Then a staircase. Then another section. Projects were changed, abandoned, rebuilt, and extended. Local decisions accumulated until the complete structure became difficult for anyone else to understand.

This is a very good picture of the risk in AI-assisted delivery.

Another Function makes sense. Another API solves today's requirement. Another queue removes a blocker. Another script automates a manual step.

Each decision can be reasonable.

Six or twelve months later, however, we may have created our own mystery house: a working system where nobody can confidently explain the whole structure.

The problem is not that every local decision was wrong.

The problem is that locally reasonable decisions can collectively create a globally unreasonable system.

## Review did not disappear, but it stopped scaling

The traditional delivery model is familiar:

**Developer → pull request → human review → deployment**

This model assumes that a person has enough time and context to reason about the change.

But what happens when generated output grows much faster than our review capacity?

Developers are moving from authoring every line toward prompting, generating, orchestrating, reviewing, and validating. A developer can remain responsible for code they did not deeply author and may not have fully read.

This leads to an uncomfortable distinction:

> **Ownership no longer guarantees understanding.**

Code review is still important. Human judgment is still important. The problem is expecting the same manual process to absorb a completely different volume of change.

We cannot solve machine-speed implementation only by asking humans to read faster.

## So, who owns the architecture?

The AI tool does not own the architecture.

It does not own the production outage, leaked data, compliance violation, unexpected Azure cost, or customer impact.

The tool vendor does not take over that responsibility either.

The developer, architect, engineering team, and ultimately the organization are still accountable.

If an AI-generated change causes a production incident, nobody will accept this as the root-cause analysis:

> "Copilot wrote it."

AI has changed implementation. It has not transferred responsibility.

The more useful question is therefore not who owns the architecture. We already know the answer.

The real question is:

> **How do we maintain architectural understanding when implementation becomes effectively unlimited?**

## Governance needs to move at the same speed

Traditional governance often depends on checkpoints: review the pull request, approve the deployment, and perhaps perform an architecture review a few times per year.

That is not enough when the system can change continuously.

This is where our thinking at DevUP has changed. Governance cannot remain a periodic control when delivery becomes continuous. It has to be part of the delivery and operational lifecycle.

The model starts to look more like this:

**Generated change → policy validation → security verification → dependency analysis → architecture visibility → compliance verification → runtime observation**

![Generated change flowing through continuous governance: policy validation, security verification, dependency analysis, architecture visibility, compliance verification, and runtime observation](/assets/images/2026/governance-pipeline.svg)

This does not mean removing humans from governance. It means using automation to preserve human oversight at a scale where inspecting every artifact manually is no longer realistic.

In practice, I think organizations need to increase their capability in six areas:

1. **Architecture visibility** — See what is actually deployed, not only what the original diagram says.
2. **Dependency mapping** — Understand how resources, systems, data, and teams are connected.
3. **Drift detection** — Notice when the running environment moves away from the intended design.
4. **Compliance verification** — Continuously validate security, policy, and regulatory requirements.
5. **Runtime observability** — Verify how the solution behaves after deployment, not only whether the pipeline was green.
6. **Lifecycle governance** — Identify ownership, outdated components, duplicated implementations, and resources that are no longer needed.

These capabilities are not only defensive. The same visibility that protects the architecture is what makes continuous optimization — performance, cost, usability — possible at all. You cannot tune what you cannot see.

We also need secure defaults and better specifications before generation starts. AI can work much more effectively when architectural intent, boundaries, and non-functional requirements are explicit.

But we should assume that change will still happen. That makes continuous visibility and verification the final safety net.

## Architecture becomes more important, not less

There is a temptation to think that architecture becomes less important when AI can generate complete implementations.

I think the opposite is true.

When implementation was expensive, cost naturally limited how much we built. When implementation becomes cheap, architectural intent becomes one of the few things preventing endless local additions.

Architecture can no longer be only a design activity before development begins. It must also become a continuous practice: checking whether the system we are operating still represents the system we intended to build.

This is also why governance becomes strategic. It is not a brake we apply after development. It is what allows the organization to benefit from AI speed without losing control of security, compliance, reliability, cost, and ownership.

At DevUP, this is the problem we spend a lot of time thinking about: not how to slow AI-assisted delivery down, but how to make architectural visibility and governance capable of keeping up. [Read more about our perspective on governing AI-generated solutions](https://www.devup.solutions/usecases/secure-ai-generated-code).

## Watch the presentations

I have explored this topic in different formats, first as **Staying in Control While AI Accelerates Integration Delivery** at AVIATORS Germany in Hamburg and later through the more direct question: **Who Owns the Architecture When AI Writes the Code?**

- [Watch: Who Owns the Architecture When AI Writes the Code?](https://www.youtube.com/watch?v=Z6vsTe_Tn28)
- [Watch: Integration Down Under — August 2026 Meeting](https://www.youtube.com/watch?v=xBvlmiX8cZM)

## Summary

AI can help us build faster, and we should use that opportunity.

But faster implementation does not automatically create better understanding. It can just as easily create more resources, dependencies, and decisions than the organization can follow.

The mistakes are not necessarily new. Their economics are.

The uncomfortable reality is that accountability never disappeared. Only visibility did.

So who owns the architecture when AI writes the code?

**We still do.**

And now we need governance that can keep up.

### If I need some assistance?

We at DevUP work with exactly this challenge: helping organizations keep architectural visibility and governance up to speed with AI-assisted delivery, using our service **Helium** for continuous insight into Azure environments.

Reach out here:

- [https://www.devup.solutions/](https://www.devup.solutions/)
- [Email: mattias@devup.solutions](mailto:mattias@devup.solutions)
- [Read more about our perspective on governing AI-generated solutions](https://www.devup.solutions/usecases/secure-ai-generated-code)
