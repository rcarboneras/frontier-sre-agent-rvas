[< Previous Solution](./Solution-19.md) | **[Home](./README.md)** | [Next Solution >](./Solution-21.md)

# Coach Guide — Challenge 20: Entra Credential Expiration Audit

> **Application focus:** Microsoft Entra ID tenant (no lab workload needed)

## Purpose

- Show that the scheduled-task pattern from Challenges 16–18 applies to the identity plane, not only Azure resources.
- Show a "fail closed" report: no success-shaped output when the data source fails.
- Expected time: 25–30 minutes (plus admin-consent lead time if the Graph permission is not pre-granted).

## Mini-Lecture (5 min before challenge)

- Name the task: `monthly-entra-credential-expiration-audit`, cron `0 7 1 * *`, mode `Autonomous`, read-only.
- Explain where credentials live: `passwordCredentials` (secrets) and `keyCredentials` (certificates) on both `applications` and `servicePrincipals`. Managed identities are excluded because Azure rotates them.
- Walk through the five buckets: EXPIRED, CRITICAL (≤7 d), EXPIRING SOON (≤30 d), EXPIRING (≤90 d), Valid.
- Stress the prerequisite: Graph application permission `Application.Read.All` on the agent managed identity, with admin consent. It is not an Azure RBAC role.

## Expected Student Output

- A PDF and an HTML report named `azure-ad-credential-expiration-audit-YYYY-MM-DD`.
- HTML with embedded data, sortable columns, search, days-remaining and status/type filters, CSV export and Entra portal links.
- Counts in the summary that match the table rows.
- A scheduled task visible in the portal, enabled, monthly.

## Common Issues and Hints

- **Symptom:** Graph returns 403. **Fix:** the managed identity lacks `Application.Read.All` or admin consent is missing. Check the app-role assignment on the identity's service principal.
- **Symptom:** Report shows only 100 apps in a large tenant. **Fix:** the agent stopped at the first page. Ask it to follow every `@odata.nextLink` and report the total objects read.
- **Symptom:** Managed identities appear in the list. **Fix:** filter out `servicePrincipalType eq 'ManagedIdentity'`.
- **Symptom:** Days-remaining sorts as text (1, 10, 2). **Fix:** ask for numeric sort on days and date sort on `endDateTime`.
- **Symptom:** Student wants the task to email owners. **Fix:** keep the task read-only and notification-free; discuss delivery as a separate, reviewed process.

## Debrief Discussion Guide

- Why exclude managed identities? → Their credentials are platform-managed and rotate automatically.
- What should happen when Graph retrieval fails? → No report that looks complete; surface the error and the missing prerequisite.
- Who acts on a CRITICAL credential, and how fast? → App owner rotates; the platform team tracks to closure.
- How would you extend this to Key Vault certificates? → Add a Resource Graph / Key Vault read, keep the same buckets and report format.

## Success Criteria Notes

- Be strict on the fail-closed behavior and on count reconciliation.
- Accept a manual run of the prompt if the student cannot create scheduled tasks in their environment, provided they can explain the task definition.
- If the tenant has very few credentials, ask the student to show the zero-or-low result is real (objects read > 0).
