[< Previous Challenge](./Challenge-19.md) — **[Home](../README.md)** — [Next Challenge >](./Challenge-21.md)

# Challenge 20 — Entra Credential Expiration Audit *(optional)*

> **Optional extension:** Identity & governance scheduled operations. Run it after the core track, or on its own with a customer that cares about identity hygiene.

> **Application focus:** Microsoft Entra ID tenant (no lab workload needed)

> **Capability**: Scheduled Tasks · Microsoft Graph · Identity Hygiene · Actionable Reporting

## Introduction

Expired client secrets and certificates are one of the most common causes of "nothing changed but it stopped working" outages. Nobody owns the expiry date until an integration fails at 3 AM.

In this challenge the agent runs a **monthly, read-only audit** of every client secret and certificate on Entra app registrations and service principals. It classifies each credential by days to expiry and publishes two artifacts: a PDF for the record and a self-contained HTML report that owners can sort, filter and export without calling Graph again.

This extends the proactive pattern from Challenges 16–18 from Azure resources to the **identity plane**.

## Description

### Before you start

- The agent's managed identity needs the Microsoft Graph **application** permission `Application.Read.All` with tenant-wide admin consent. This is a Graph app-role assignment, not an Azure RBAC role. Agree with the tenant admin who grants it.
- Review the task definition in `Resources/azure-sre-agent-config/automations/scheduled-tasks/monthly-entra-credential-expiration-audit.yaml`.

### What to do

- Run the audit on demand by giving the agent the prompt from the task definition, and review both artifacts.
- Check that managed identities are excluded and that every page of results was retrieved, not just the first.
- In the HTML report, filter to credentials expiring within 30 days, sort by days remaining, and export the filtered list as CSV.
- Create the scheduled task in your agent (portal or data-plane API) so it runs monthly, then trigger it once manually.
- Decide who should receive the report and what an owner does with a **CRITICAL** item. Do not add email delivery to the task; describe the process instead.

## Success Criteria

1. The agent produces both `azure-ad-credential-expiration-audit-YYYY-MM-DD.pdf` and the matching `.html` report from live Graph data.
2. Every credential is classified as EXPIRED, CRITICAL, EXPIRING SOON, EXPIRING or Valid, and the report counts match the rows in the table.
3. The HTML report supports search, days-remaining filters, sorting on every column in both directions, and filtered CSV export.
4. The scheduled task exists, is enabled, runs monthly and is read-only.
5. **Explain to your coach** — why the task must refuse to publish a report when Graph retrieval fails, and what would go wrong if it reported "0 expiring credentials" after a 403.

## Learning Resources

- [Azure SRE Agent — scheduled tasks](https://learn.microsoft.com/en-us/azure/sre-agent/scheduled-tasks)
- [Microsoft Graph — application resource type](https://learn.microsoft.com/en-us/graph/api/resources/application)
- [Microsoft Graph — passwordCredential resource type](https://learn.microsoft.com/en-us/graph/api/resources/passwordcredential)
- [Microsoft Graph — paging](https://learn.microsoft.com/en-us/graph/paging)
- [Grant an app role to a managed identity](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/how-to-assign-app-role-managed-identity)

## Tips

- An empty report is not a clean tenant. Ask the agent how many applications and service principals it read before you trust a zero.
- Certificates (`keyCredentials`) and secrets (`passwordCredentials`) live on both the application and the service principal objects. Check that both are covered.
- The HTML report embeds its data, so it can be shared with app owners who have no Graph access.
