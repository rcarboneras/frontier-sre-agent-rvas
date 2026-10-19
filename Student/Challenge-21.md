[< Previous Challenge](./Challenge-20.md) — **[Home](../README.md)** — [Next Challenge >](./Challenge-22.md)

# Challenge 21 — Orphaned RBAC Principal Audit *(optional)*

> **Optional extension:** Identity & governance scheduled operations. Run it after the core track, or on its own with a customer that cares about access governance.

> **Application focus:** Azure RBAC across management groups, subscriptions and resource groups

> **Capability**: Scheduled Tasks · Azure RBAC · Microsoft Graph · Coverage-Aware Reporting

## Introduction

When a user leaves, a group is deleted or an app registration is removed, its Azure role assignments stay behind as "Identity not found". They clutter access reviews, hide real exposure and can break automation that expects every assignment to resolve.

In this challenge the agent runs a **monthly, read-only audit** of RBAC assignments at every scope it can read, verifies each principal against Microsoft Graph, and reports only the assignments whose principal is **confirmed** deleted. Anything it could not verify is reported as a coverage gap, not as a finding.

## Description

### Before you start

- The agent's managed identity needs **Reader** on the management groups or subscriptions you want to audit, and the Microsoft Graph **application** permission `Directory.Read.All` with admin consent.
- Review the task definition in [`Resources/azure-sre-agent-config/automations/scheduled-tasks/monthly-orphaned-rbac-principal-audit.yaml`](./Resources/azure-sre-agent-config/automations/scheduled-tasks/monthly-orphaned-rbac-principal-audit.yaml).
- Optional: create a test orphan in a lab resource group by assigning Reader to a test user or app registration and then deleting that principal.

### What to do

<details>
<summary>Prompt to use (same as the task definition)</summary>

```text
Produce a read-only, complete audit of Azure RBAC assignments at management-group, subscription,
resource-group and tenant-root scopes whose assigned Entra principal no longer resolves. Include
orphaned Users, Groups and Service Principals; do not modify assignments. Prerequisite: the agent
user-assigned managed identity needs the Microsoft Graph application permission Directory.Read.All
with admin consent; acquire Graph tokens for https://graph.microsoft.com/.default with that
identity. If Graph returns 401, 403, consent or permission errors, do not classify any principal as
deleted; still create all artifacts, mark the audit as partial coverage and report the exact missing
prerequisite. Scope: enumerate every accessible management group, subscription and resource group
plus readable tenant-root assignments; collect direct and inherited assignments where the APIs
expose them. Verification: de-duplicate principal IDs and verify them with Microsoft Graph
directoryObjects/getByIds; treat a principal as orphaned only when the directory lookup confirms it
does not exist, never from a missing display name, an authorization failure or an incomplete
Resource Graph result. Keep scope, scope type, principal ID, principal type, role and assignment ID
for every confirmed orphan. Resilience: retry 408, 429 and 5xx up to three times with 2, 5 and 10
second backoff; continue past failed scopes and report them as coverage warnings. Artifacts:
UTC-timestamped CSV, Markdown and self-contained HTML reports; the HTML has summary metrics,
free-text search, scope-type and principal-type filters, Azure portal links to each scope and
assignment, a Copy principal action, and keyboard-accessible sortable columns with aria-sort and a
deterministic assignment-ID tie-breaker. Return download links and a count summary by principal type
and scope type. Safety: read-only; never delete, modify or remediate RBAC assignments. If nothing is
found, still produce the three artifacts with headers and an explicit zero-result summary. Clearly
distinguish complete coverage from partial coverage.
```

</details>

- Run the audit on demand using the prompt above, and review the CSV, Markdown and HTML artifacts.
- Compare the findings with the portal's role assignments view ("Identity not found") for at least one scope.
- Remove the Graph permission from a test identity (or run with an identity that lacks it) and observe how the report changes. Restore it afterwards.
- Create the scheduled task in your agent so it runs monthly, then trigger it once manually.
- Agree who reviews the findings and how a confirmed orphan is removed. The agent must never remove it.

## Success Criteria

1. The agent produces CSV, Markdown and HTML artifacts with a count summary by principal type and scope type.
2. Every reported orphan was confirmed missing through a Graph directory lookup, with scope, principal ID, principal type, role and assignment ID.
3. Unreadable scopes and Graph permission failures appear as coverage warnings, and the report states whether coverage is complete or partial.
4. The scheduled task exists, is enabled, runs monthly and makes no changes to role assignments.
5. **Explain to your coach** — why "the display name is empty" is not proof that a principal was deleted, and why this task reports and does not remediate.

## Learning Resources

- [Azure SRE Agent — scheduled tasks](https://learn.microsoft.com/en-us/azure/sre-agent/scheduled-tasks)
- [Troubleshoot Azure RBAC — "Identity not found"](https://learn.microsoft.com/en-us/azure/role-based-access-control/troubleshooting#symptom---role-assignments-with-identity-not-found)
- [List Azure role assignments](https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-rest)
- [Microsoft Graph — directoryObject: getByIds](https://learn.microsoft.com/en-us/graph/api/directoryobject-getbyids)
- [Azure Resource Graph — authorization resources](https://learn.microsoft.com/en-us/azure/governance/resource-graph/reference/supported-tables-resources#authorizationresources)

## Tips

- Deleted principals can take a few minutes to disappear from Graph. A brand-new test orphan may not show up straight away.
- Inherited assignments appear at every child scope. De-duplicate by assignment ID before counting.
- A zero-result report is still a deliverable: the artifacts should exist, with headers and an explicit "no orphaned assignments found" summary.
