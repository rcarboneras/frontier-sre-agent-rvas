[< Previous Solution](./Solution-21.md) | **[Home](./README.md)**

# Coach Guide — Challenge 22: Tenant License Optimization

> **Application focus:** Microsoft Entra ID tenant licensing (through a custom MCP connector)

## Purpose

- Show how to extend the agent with a custom MCP server when a first-party server cannot be connected directly.
- Show managed-identity-to-managed-identity trust with no stored secrets and read-only Graph access.
- Apply the FinOps mindset of Challenge 18 to license spend.
- Expected time: 45–60 minutes including bridge deployment (20–30 minutes if the bridge is pre-deployed).

## Mini-Lecture (10 min before challenge)

- Why a bridge: Microsoft MCP Server for Enterprise requires a pre-registered client and does not support Dynamic Client Registration; the agent's OAuth MCP connector requires DCR. The bridge exposes a compatible Streamable-HTTP endpoint.
- Flow: SRE Agent → managed identity token → Container Apps bridge (validates JWT and `McpBridge.Access` role) → bridge managed identity → Microsoft Graph (read-only).
- Connector settings: name `license-optimization-mcp`, Streamable-HTTP, managed identity, scope `api://<bridge-app-client-id>/.default`; the bridge `BRIDGE_AUDIENCE` must be `api://<bridge-app-client-id>`.
- Tools to highlight: `get_tenant_license_consumption`, `find_unassigned_licenses`, `detect_inactive_licensed_users`, `generate_license_optimization_recommendations`.
- Task: `weekly-tenant-license-optimization`, cron `0 9 * * 1`, mode `Autonomous`, read-only.

## Expected Student Output

- Connector shows as connected and its tools are selectable.
- A table of SKUs with purchased/enabled, consumed, unassigned and utilization %, with SKUs under 70 % highlighted.
- A list of licensed users inactive for 90+ days grouped by `accountEnabled`, with review priority.
- Recommendations with validation steps before reclaiming, and a weekly scheduled task that produces an executive summary plus a detailed table.

## Common Issues and Hints

- **Symptom:** Connector returns 401. **Fix:** audience/scope mismatch, or the agent identity was not granted the bridge app role (script `05-grant-sre-agent-access-to-bridge-api.ps1`).
- **Symptom:** Bridge returns 403 from Graph. **Fix:** Graph application permissions were not granted or consented for the bridge managed identity (script `04-grant-graph-permissions-to-bridge-mi.ps1`).
- **Symptom:** Inactive-user detection returns nothing or errors. **Fix:** `signInActivity` needs Entra ID P1/P2 and audit-log read permission.
- **Symptom:** Container App fails to pull the image. **Fix:** the bridge identity needs AcrPull on the registry; rerun the deployment script.
- **Symptom:** Student asks the agent to remove licenses. **Fix:** the bridge is read-only by design; reclamation is a reviewed admin action.

## Debrief Discussion Guide

- Why not give the SRE Agent Graph permissions directly? → Least privilege; the bridge narrows the surface to curated read-only tools.
- What is the value of purpose-built tools over a generic `graph_get`? → Deterministic calculations, fewer tokens, safer prompts.
- Which validation steps should come before reclaiming a license? → Leave/long absence, shared or service accounts, licenses required by policy, owner confirmation.
- How would you show business value to the customer? → Unassigned units × SKU price as cost-avoidance at renewal.

## Success Criteria Notes

- Pre-deploy the bridge for short events; the learning goal is the connector, tools and reporting, not the deployment scripts.
- Accept a tenant without P1/P2 if the student explains why inactivity data is unavailable and the report says so.
- Be strict that nothing in the flow modifies licenses or users.
