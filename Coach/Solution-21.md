[< Previous Solution](./Solution-20.md) | **[Home](./README.md)** | [Next Solution >](./Solution-22.md)

# Coach Guide — Challenge 21: Orphaned RBAC Principal Audit

> **Application focus:** Azure RBAC across management groups, subscriptions and resource groups

## Purpose

- Teach evidence discipline: only a confirmed directory lookup makes a principal "orphaned".
- Show coverage-aware reporting: partial coverage is reported as partial, never as clean.
- Expected time: 25–35 minutes.

## Mini-Lecture (5–10 min before challenge)

- Name the task: `monthly-orphaned-rbac-principal-audit`, cron `0 8 1 * *`, mode `Autonomous`, read-only.
- Explain "Identity not found": the role assignment survives when its user, group or service principal is deleted.
- Two permission planes: Azure **Reader** at the scopes to audit, and Graph **Directory.Read.All** (application permission with admin consent) to verify principals.
- Verification rule: bulk `directoryObjects/getByIds`; a principal is orphaned only if Graph confirms it does not exist. Authorization errors are coverage gaps, not findings.

## Expected Student Output

- CSV, Markdown and HTML artifacts with a UTC timestamp in the filename.
- Per orphan: scope, scope type, principal ID, principal type, role, assignment ID.
- A count summary by principal type and scope type, and a clear complete/partial coverage statement.
- A zero-result run still produces all three artifacts with headers.

## Common Issues and Hints

- **Symptom:** Every assignment is reported as orphaned. **Fix:** Graph calls are failing (403). The task must mark partial coverage instead; check `Directory.Read.All` and admin consent.
- **Symptom:** Same orphan counted many times. **Fix:** inherited assignments appear at child scopes; de-duplicate by assignment ID.
- **Symptom:** Management-group or tenant-root scopes are missing. **Fix:** the identity lacks Reader at that level; this must appear as a coverage warning.
- **Symptom:** A freshly deleted test principal is not detected. **Fix:** directory replication can lag; rerun after a few minutes.
- **Symptom:** Student asks the agent to delete the orphans. **Fix:** out of scope by design; removal goes through the customer's change process.

## Debrief Discussion Guide

- Why is an empty display name not evidence of deletion? → It can be a permission or lookup failure.
- What is the risk of auto-remediating orphans? → Deleting an assignment that was only unreadable breaks access; irreversible without records.
- How does this help access reviews? → Removes noise so reviewers focus on real principals.
- What would you add next? → Trend month over month; tag owner per subscription.

## Success Criteria Notes

- Be strict on confirmed-only classification and explicit coverage status.
- Accept a single-subscription scope in small labs, as long as the student explains how scope expands with Reader at management-group level.
- The permission-removal experiment is recommended, not mandatory, if the student cannot change Graph permissions.
