---
layout: post
title: "Identity Is Not the Only Perimeter"
description: "We built an MCP server for Helium with one rule: data access must be the same as in the portal. That is one boundary. An AI agent has a path per connection, and every path needs its own: what we learned, and where the Azure controls fit."
date: 2026-10-15 09:00:00 +0200
categories: [Helium, AI]
tags: [AI, Security, Azure, MCP, Network Security Perimeter, Microsoft Foundry, API Management, Governance]
author: "Mattias Lögdberg"
image: /assets/images/2026/agent-boundaries-hero.png
comments: true
---

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/agent-boundaries-hero-mobile.svg">
  <img src="/assets/images/2026/agent-boundaries-hero.svg" alt="An invoice agent with its own identity reaches Azure Storage, an MCP server, and a Logic App through a network boundary, with a boundary marker on each path. A dashed red path leaves the approved, allowlisted MCP server and crosses the boundary to an attacker endpoint. Headline: Identity is not the only perimeter. Every path needs a boundary.">
</picture>

Earlier this year we built an MCP server for Helium. It runs as an Azure Functions app, users sign in with their Entra account, and an AI assistant can ask it about the Azure environments that user already has access to. It is available to every Helium customer today.

It exposes ten tools. All of them are read-only, and all of them show a user exactly what the portal would show them. Nothing more.

Those were deliberate boundaries. I thought they were the boundary.

This is the third of four articles in this series, and it is about boundaries. In [Every AI Agent Needs an Identity — and an Owner]({% post_url 2026-10-01-every-ai-agent-needs-an-identity %}), I argued that every agent needs its own identity, a clear authority model, and a named owner.

Identity gives us the context. But it only answers one question:

> Who is allowed to ask?

It does not answer where the agent can connect, which tools it can reach, or where the data can go.

## Exfiltration: Every action was allowed

Microsoft's incident responders recently [walked through an attack pattern against an invoice-processing agent](https://www.microsoft.com/en-us/security/blog/2026/06/30/securing-ai-agents-ai-tools-move-from-reading-acting/). The scenario is illustrative, but they write that the technique has been observed against enterprise agents in 2026.

An attacker changed the tool description on an approved third-party MCP server. The hidden instruction told the agent to summarize the last thirty unpaid invoices and attach the summary to an ordinary tool call.

The agent did exactly that. The server returned a normal response and passed the data on to the attacker.

The part I keep coming back to is Microsoft's own summary. The tool was approved. The data query used the analyst's own permissions. The outbound call went to a server that was on the allowlist.

Identity worked. Least privilege worked. The allowlist worked.

The data still left.

> An allowlisted destination is still a destination.

## Access: Two boundaries, one rule

Read that scenario with our server in mind. An MCP server is a new door into customer data. Two boundaries decide what that door means.

The first is **MCP access**: who is allowed to connect at all. For us that is an Entra sign-in. No sign-in, no tools.

The second is **data access**: what a signed-in user can see once connected. This is the one that matters, and the rule we set for it is simple.

> Data access has to be the same whether you use the portal or the MCP server.

The MCP server holds no data of its own and no permissions of its own. It takes the user's token, exchanges it for a token to the Helium backend on the user's behalf, and calls the same API the portal calls. The backend decides what the user can see. If you cannot see an environment in the portal, the tool that lists environments returns nothing.

This sounds obvious. It is easy to get wrong. The shortcut is to give the server its own identity with broad read access and let the tools filter. Then the server has become a second authorization system, and the two will drift apart. Every MCP client would be trusting our filtering code instead of the customer's access model.

Read-only is the supporting boundary. No tool calls a write endpoint, so an agent cannot change a customer's environment through us, whatever a description tells it to do.

What read-only does not do is limit where the answer goes. Every tool returns data: findings, resource names, failed checks, remediation guidance. The agent decides where that data goes next. Into a summary. Into another tool call. Into a parameter on a request to a server we have never heard of.

> Read-only limits what the server can do. It does not limit what the agent does with the answer.

The data does not leave through our server. It leaves through the client the user chose to connect. We rolled the server out as a preview to the customers we knew needed it, then to everyone, and we did not restrict which clients a user can connect. The user decides that. So the user's client is part of the customer's boundary, not ours.

That was the first thing building an MCP server taught me about boundaries. Data access is a boundary we own, and it has to be the same in every channel. The path the data takes afterwards is a boundary someone else owns.

## Boundaries: The other half of the firewall story

In the [governance article]({% post_url 2026-09-23-governance-needs-to-move-at-the-same-speed-as-ai %}), I wrote that guardrails provide boundaries and governance provides direction. I also mentioned a [DevUP Talks conversation with Simon Wåhlin](https://youtu.be/-eAswM6jzxA) about an AI that solved a connectivity problem by removing the firewall in the way.

That was an agent removing a boundary.

This article is about the opposite problem. The boundary is in place. The identity is correct. And the data leaves anyway, through a path nobody thought of as a path.

## Paths: An agent has more than one way out

We are used to thinking about one perimeter per workload. A virtual network, a firewall, a set of private endpoints.

An agent does not fit that picture. It reads data, calls tools, calls other agents, and sends results somewhere. Each of those is a path. Each path can carry data.

This is the same example as in the identity article. Last time, every connection carried an identity. This time, every connection also needs a boundary.

<picture>
  <source media="(max-width: 600px)" srcset="/assets/images/2026/agent-boundaries-chain-mobile.svg">
  <img src="/assets/images/2026/agent-boundaries-chain.svg" alt="An AI agent connected to Azure Storage, an MCP server, a Logic App, and a customer system. Every connection carries an identity marker and a boundary marker: perimeter rules, a gateway for tools, a private endpoint, and an egress allowlist.">
</picture>

Azure has boundary controls for those paths. But they are separate controls, and each covers a different part of the picture.

**Around the data.** [Network Security Perimeter](https://learn.microsoft.com/en-us/azure/private-link/network-security-perimeter-concepts) puts a logical boundary around PaaS resources such as Storage, Key Vault, AI Search and Foundry. In enforced mode, public traffic in and out is denied unless a rule allows it. Outbound rules are written per FQDN, and there are access logs.

**Around the agent runtime.** Foundry agents can run with [public egress, in your own virtual network, or in a managed virtual network](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/networking-options). A private endpoint on the Foundry resource only protects the inbound side. With public egress, the agent can still reach anything on the internet. For hosted agents, there are also [network egress rules](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/add-hosted-agent-guardrails) that allow or deny outbound calls per host. At the time of writing they are in preview.

**Around the tools.** MCP traffic from Foundry agents can be [routed through an AI gateway](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/governance) in API Management, where rate limits, IP filters and logging apply. Also preview, and it only applies to tools created after the gateway was connected. Existing tools keep calling the MCP server directly.

Three controls. Three different places where an agent can run or send data.

None of them covers all of it.

> An agent does not have one perimeter. It has one path per connection, and every path needs its own boundary.

## Coverage: Isolated does not mean private

The gaps are not always where you expect.

In a network-isolated Foundry setup, some tools still [use public endpoints](https://learn.microsoft.com/en-us/azure/foundry/how-to/configure-private-link#agent-tools-with-network-isolation). Bing grounding, web search and SharePoint grounding work, but their traffic goes over the public internet. If the requirement is that everything stays private, those tools need to be blocked, not assumed away.

The gateway has its own version of the identity problem. In the preview [AI Gateway tier of API Management](https://learn.microsoft.com/en-us/azure/api-management/ai-gateway-overview), applications authenticate with a runtime access key, and that key reaches every model and tool on the gateway. The boundary component itself runs on a shared static key. That is exactly what the identity article warned against.

And Network Security Perimeter has sharp edges that a real rollout will hit. SAS tokens are rejected for traffic inside the perimeter. It is not supported on Log Analytics workspaces enabled for Sentinel. Azure Backup is not supported on storage accounts in a perimeter.

None of this is a reason to wait. It is a reason to know which paths are covered, and by what.

## Configuration: The description is the instruction

The attack in the invoice scenario did not change a firewall rule. It changed a tool description.

We learned how much weight a description carries from the harmless direction. Our server has two tools that overlap: one lists findings across a whole environment, one returns everything to fix on a single resource. Assistants kept picking the environment-wide tool for single-resource questions, then calling the detail tool once per finding to answer a question about one resource.

We did not change any code. We rewrote the descriptions. The environment-wide tools now say, in their descriptions, that for a single resource you should use the other one. That was the fix we shipped, and nothing else changed.

> A tool description is not documentation. It is the instruction the model follows.

The same lever that fixed our tool selection is the lever the attacker used.

So we treat the tool catalog as code. It lives in the repository as one markdown file, the single source of truth for which tools exist. A test reflects over the assembly and fails when a tool is registered without a row in the catalog, or a row exists without a tool. Any add, rename or removal has to ship an update to the customer tutorial in the same change. The wording of a description is reviewed in the pull request like any other change.

That is also Microsoft's recommendation: keep an allowlist of approved MCP publishers and servers, enable only the tools an agent needs, and review changes to MCP configuration like changes to production code.

A network rule is infrastructure. A tool description is text.

> Both decide where the data ends up.

## Rollout: Observe first, then enforce

The boundary products share one good design choice. They all start by observing.

Network Security Perimeter starts in transition mode. It logs what would be denied before anything is blocked. Foundry egress rules have an audit mode that does the same for hosted agents. Microsoft recommends both before you enforce.

This is the same order as the governance loop. Discover what is actually happening. Understand it. Then decide what to block.

An agent's real dependencies are rarely the ones on the diagram. Package managers follow redirects. Tools call other hosts. An allowlist you have not observed first will block something the agent actually needs.

## Controls: Seven practical places to start

1. **Map the paths per agent.** Data it reads, tools it calls, agents it calls, and where results are sent. Include the client. That is a path too.
2. **One data boundary for every channel.** An MCP server, an API and a portal that reach the same data must enforce the same access. Never give the server its own broad identity and filter in the tools.
3. **Put a boundary on each path.** A perimeter around the data, egress control on the runtime, a gateway in front of the tools.
4. **Start in audit mode.** Transition mode for the perimeter, audit mode for egress rules. Read the logs before enforcing.
5. **Block the public-endpoint tools you do not need.** Isolated does not mean private.
6. **Treat MCP configuration as code.** Approved servers, scoped tools, a catalog that a test enforces, reviewed descriptions.
7. **Re-check coverage.** Tools created before the gateway was connected, and settings changed during troubleshooting, are where the gaps appear.

## Governance: Boundaries drift too

A boundary is a configuration. Configurations drift.

A public endpoint is enabled for troubleshooting and never disabled. A tool is added before the gateway is connected. A resource is created outside the perimeter because the deadline was tomorrow.

This is how we think about it in Helium, our Continuous Cloud Governance Platform for Azure. It is the same loop as for identity:

> **Discover → Understand → Prioritize → Improve → Verify**

A public endpoint is a finding. A public endpoint on the storage account an agent reads from, with no perimeter and no private endpoint, is a priority.

The MCP server is how that picture reaches an AI assistant. The loop is what keeps the picture true.

## Accountability: So, who owns the boundary?

The answer has not changed:

> We still do.

The agent does not decide which paths exist. The tool vendor does not decide what leaves our environment. We do, whether we decided it on purpose or by leaving a path open.

I decided that our server would show a user exactly what the portal shows them, and nothing more. I did not decide where its answers go. That second decision was made for me, by every client a user connects. Owning the boundary means knowing which of those decisions you made and which ones you did not.

## Validation: Boundaries limit, they do not prove

A boundary limits what could happen.

It does not tell you what did happen.

In the invoice scenario, every control did its job and the data still left through an allowed path. The only way to find that is to reconstruct what the agent actually did: which tool, which parameters, which identity, which destination.

That is where the last article in this series begins. After identity and boundaries, we return to validation: what could the agent do, what did it do, and can we prove the complete chain?

---

**Want to know which of your Azure resources are actually behind a boundary?**

DevUP Helium is our Continuous Cloud Governance Platform for Azure. [Learn more about Helium](https://www.devup.solutions/) or [contact Mattias](mailto:mattias@devup.solutions).
