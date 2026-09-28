---
layout: post
title: "Identity Is Not the Only Perimeter"
description: "Identity decides who is allowed to ask. Boundaries decide where the data can go. An AI agent has no single perimeter: each path it can take needs its own boundary, and someone needs to know which paths are covered."
date: 2026-10-15 09:00:00 +0200
categories: [Helium, AI]
tags: [AI, Security, Azure, Network Security Perimeter, Microsoft Foundry, MCP, API Management, Governance]
author: "Mattias Lögdberg"
comments: true
---

<!-- TODO(Mattias): hero image (desktop, mobile, 1200x630 PNG) in the same style as the identity post, then add `image:` to the front matter. -->

In [Every AI Agent Needs an Identity — and an Owner]({% post_url 2026-10-01-every-ai-agent-needs-an-identity %}), I argued that every agent needs its own identity, a clear authority model, and a named owner.

This is the third of four articles in this series. It is about boundaries.

Identity decides who is allowed to ask.

It does not decide where the data can go.

## Exfiltration: Every action was allowed

Microsoft's incident responders recently [walked through an attack pattern against an invoice-processing agent](https://www.microsoft.com/en-us/security/blog/2026/06/30/securing-ai-agents-ai-tools-move-from-reading-acting/). The scenario is illustrative, but they write that the technique has been observed against enterprise agents in 2026. An attacker changed the tool description on an approved third-party MCP server. The hidden instruction told the agent to summarize the last thirty unpaid invoices and attach the summary to an ordinary tool call.

The agent did exactly that. The server returned a normal response and passed the data on to the attacker.

The part I keep coming back to is Microsoft's own summary. The tool was approved. The data query used the analyst's own permissions. The outbound call went to a server that was on the allowlist.

Identity worked. Least privilege worked. The allowlist worked.

The data still left.

## Paths: An agent has more than one way out

We are used to thinking about one perimeter per workload. A virtual network, a firewall, a set of private endpoints.

An agent does not fit that picture. It reads data, calls tools, calls other agents, and sends results somewhere. Each of those is a path. Each path can carry data.

Azure now has boundary controls for most of those paths. But they are separate controls, and each covers a different part of the picture.

**Around the data.** [Network Security Perimeter](https://learn.microsoft.com/en-us/azure/private-link/network-security-perimeter-concepts) puts a logical boundary around PaaS resources such as Storage, Key Vault, AI Search and Foundry. In enforced mode, public traffic in and out is denied unless a rule allows it. Outbound rules are written per FQDN, and there are access logs.

**Around the agent runtime.** Foundry agents can run with [public egress, in your own virtual network, or in a managed virtual network](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/networking-options). A private endpoint on the Foundry resource only protects the inbound side. With public egress, the agent can still reach anything on the internet. For hosted agents, there are also [network egress rules](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/add-hosted-agent-guardrails) that allow or deny outbound calls per host. They are in preview.

**Around the tools.** MCP traffic from Foundry agents can be [routed through an AI gateway](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/governance) in API Management, where rate limits, IP filters and logging apply. It is in preview, and only applies to tools created after the gateway was connected. Existing tools keep calling the MCP server directly.

**Around the device.** For agents and MCP clients on a managed device, the [Global Secure Access MCP firewall](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-configure-mcp-firewall) can allow or block MCP servers and individual tools. It is in preview too. It needs the Global Secure Access client and TLS inspection, and it only sees remote MCP servers.

Four controls. Four different places where an agent can run or send data.

None of them covers all of it.

## Coverage: Isolated does not mean private

The gaps are not always where you expect.

In a network-isolated Foundry setup, some tools still [use public endpoints](https://learn.microsoft.com/en-us/azure/foundry/how-to/configure-private-link#agent-tools-with-network-isolation). Bing grounding, web search and SharePoint grounding work, but their traffic goes over the public internet. If the requirement is that everything stays private, those tools need to be blocked, not just assumed away.

The gateway has its own version of the identity problem. In the preview [AI Gateway tier of API Management](https://learn.microsoft.com/en-us/azure/api-management/ai-gateway-overview), applications authenticate with a runtime access key, and that key reaches every model and tool on the gateway. The boundary component itself runs on a shared static key. That is exactly what the identity article warned against.

And Network Security Perimeter has sharp edges that a real rollout will hit. SAS tokens are rejected for traffic inside the perimeter. It is not supported on Log Analytics workspaces enabled for Sentinel. Azure Backup is not supported on storage accounts in a perimeter.

<!-- TODO(Mattias): a real example from integration or Helium work where a boundary looked closed on the diagram but one path was still open (for example a public endpoint left on after troubleshooting, or a tool that bypassed the private network). -->

None of this is a reason to wait. It is a reason to know which paths are covered, and by what.

## Rollout: Observe first, then enforce

The boundary products share one good design choice. They all start by observing.

Network Security Perimeter starts in transition mode. It logs what would be denied before anything is blocked. Foundry egress rules have an audit mode that does the same for hosted agents. Microsoft recommends both before you enforce.

This is the same order as the governance loop from the [governance article]({% post_url 2026-09-23-governance-needs-to-move-at-the-same-speed-as-ai %}). Discover what is actually happening. Understand it. Then decide what to block.

An agent's real dependencies are rarely the ones on the diagram. Package managers follow redirects. Tools call other hosts. Enforcing an allowlist you have not observed first is how you break production on a Friday.

## Configuration: Tool descriptions are part of the perimeter

The attack in the invoice scenario did not change a firewall rule. It changed a tool description.

Microsoft's recommendation is to treat tool descriptions like system prompts. Keep an allowlist of approved MCP publishers and servers. Enable only the tools an agent needs, instead of everything a server exposes. Review changes to MCP configuration like changes to production code.

That is the boundary most teams do not see yet. A network rule is infrastructure. A tool description is text. Both decide where data ends up.

## Controls: Where I would start

1. **Map the paths per agent.** Data it reads, tools it calls, agents it calls, and where results are sent.
2. **Put a boundary on each path.** Perimeter around the data, egress control on the runtime, a gateway in front of the tools.
3. **Start in audit mode.** Transition mode for the perimeter, audit mode for egress rules. Read the logs before enforcing.
4. **Block the public-endpoint tools you do not need.** Isolated does not mean private.
5. **Treat MCP configuration as code.** Approved servers, scoped tools, reviewed changes.
6. **Re-check coverage.** Tools created before the gateway was connected, and settings changed during troubleshooting, are where the gaps appear.

## Validation: Boundaries limit, they do not prove

A boundary limits what could happen.

It does not tell you what did happen.

In the invoice scenario, every control did its job and the data still left through an allowed path. The only way to find that is to reconstruct what the agent actually did: which tool, which parameters, which identity, which destination.

That is the last article in this series. After identity and boundaries, we return to validation: what could the agent do, what did it do, and can we prove the complete chain?

---

**Want to know which of your Azure resources are actually behind a boundary?**

DevUP Helium is our Continuous Cloud Governance Platform for Azure. [Learn more about Helium](https://www.devup.solutions/) or [contact Mattias](mailto:mattias@devup.solutions).
