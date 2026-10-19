[< Previous Challenge](./Challenge-21.md) — **[Home](../README.md)**

# Challenge 22 — Tenant License Optimization *(optional)*

> **Optional extension:** Identity & governance scheduled operations. Run it with a customer that wants to reclaim unused Microsoft 365 / Entra licenses.

> **Application focus:** Microsoft Entra ID tenant licensing (through a custom MCP connector)

> **Capability**: Custom MCP Connector · Managed Identity · Microsoft Graph · FinOps for Licensing · Scheduled Tasks

## Introduction

License spend is often bigger than the Azure bill it sits next to, and nobody reviews it until renewal. Unassigned seats, SKUs at 40 % utilization and licensed users who have not signed in for months are all money on the table.

Azure SRE Agent cannot connect directly to Microsoft MCP Server for Enterprise today, because that server needs a pre-registered client and the agent's OAuth connector needs Dynamic Client Registration. The **SRE Agent MCP Enterprise Bridge** closes that gap: a small Container Apps MCP server that validates the agent's managed identity token and calls Microsoft Graph read-only with its own managed identity. No secrets are stored and the agent never gets Graph permissions itself.

In this challenge you connect the bridge as `license-optimization-mcp`, investigate the tenant interactively, and then schedule a weekly license optimization report.

## Description

### Before you start

- Deploy the bridge by following the [SRE Agent MCP Enterprise Bridge](https://github.com/rcarboneras/SRE-Agent/tree/main/SRE-MCPEnterprise-bridge) README. You need permission to create an app registration, grant Graph application permissions with admin consent, and deploy Azure Container Apps and Azure Container Registry.
- Review the task definition in `Resources/azure-sre-agent-config/automations/scheduled-tasks/weekly-tenant-license-optimization.yaml`.

### What to do

- Add the bridge to your agent as a **Streamable-HTTP** MCP connector named `license-optimization-mcp`, authenticated with the agent's managed identity and the bridge API scope. Select its tools.
- Ask the agent for current license consumption by SKU and highlight SKUs below 70 % utilization.
- Ask it to find licensed users with no sign-in in the last 90 days, grouped by whether the account is enabled, and to recommend which to review first.
- Ask it for a short list of license optimization recommendations, including the checks needed before reclaiming any license.
- Create the weekly scheduled task and trigger it once manually.

## Success Criteria

1. The `license-optimization-mcp` connector is connected with managed identity authentication and its tools are available to the agent.
2. The agent returns license consumption by SKU with purchased, consumed, unassigned and utilization values, and flags SKUs under 70 %.
3. The agent lists inactive licensed users (90 days) and produces prioritized, read-only optimization recommendations with validation steps.
4. The weekly scheduled task exists, is enabled and produces an executive summary followed by a detailed table.
5. **Explain to your coach** — how the bridge keeps the agent away from direct Graph permissions, and why a bridge is needed instead of connecting the agent to Microsoft MCP Server for Enterprise directly.

## Learning Resources

- [Azure SRE Agent — scheduled tasks](https://learn.microsoft.com/en-us/azure/sre-agent/scheduled-tasks)
- [Microsoft MCP Server for Enterprise](https://github.com/mcp/microsoft/EnterpriseMCP)
- [Microsoft Graph — subscribedSku resource type](https://learn.microsoft.com/en-us/graph/api/resources/subscribedsku)
- [Microsoft Graph — signInActivity](https://learn.microsoft.com/en-us/graph/api/resources/signinactivity)
- [Azure Container Apps — managed identities](https://learn.microsoft.com/en-us/azure/container-apps/managed-identity)

## Tips

- The token scope in the connector and the bridge's expected audience must match exactly (`api://<bridge-app-client-id>/.default` and `api://<bridge-app-client-id>`). A mismatch shows up as 401 from the bridge.
- Sign-in activity needs an Entra ID P1 or P2 license in the tenant. Without it, inactivity cannot be measured.
- The bridge scales to zero, so the first call after a quiet period can take a few seconds longer.
