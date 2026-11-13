# Worked example: right-sizing an SRE Agent managed identity

Sanitized, illustrative adaptation of the upstream example. Replace placeholders and gather fresh evidence before using it; the observations below are not evidence about your environment.

**Identity:** user-assigned MI `<MANAGED_IDENTITY_NAME>`
(object ID `<IDENTITY_OBJECT_ID>`, client ID `<IDENTITY_CLIENT_ID>`)

**Current subscription-level roles:** Owner, Reader, Monitoring Contributor
(plus narrow roles: Grafana Viewer, Automation Contributor on `<AUTOMATION_ACCOUNT>`,
Azure Connected Machine Resource Manager on `<ARC_MACHINE>`, Monitoring Contributor on `<AGENT_RESOURCE_GROUP>`).

## Evidence (Activity Log, caller = MI object ID, status Succeeded)
Illustrative coverage: 60 days returned successfully; an earlier slice timed out and is **not covered**.

| Operation | Scope | Covered by existing role? |
|---|---|---|
| `Microsoft.AlertsManagement/alerts/changestate/action` | In-scope VMs / Arc machines | Yes: Monitoring Contributor (`Microsoft.AlertsManagement/alerts/*`) |
| `Microsoft.Insights/dataCollectionRules/write` | `<RESOURCE_GROUP>/<DATA_COLLECTION_RULE>` | Yes: Monitoring Contributor (`Microsoft.Insights/dataCollectionRules/*`) |
| `Microsoft.Compute/virtualMachines/start/action` | `<RESOURCE_GROUP>/<VM_NAME>` | **No**: only Owner covers it today |
| `Microsoft.Resources/subscriptions/resourcegroups/write` | `<TEST_RESOURCE_GROUP>` | Confirmed one-off test build; not a standing need |
| `Microsoft.Management/register/action` | Provider registration | Confirmed one-off setup; not a standing need |

Reads are never in the Activity Log; Reader + Monitoring Contributor (`*/read`) already cover them.

## Result
- **Recommend removing** Owner on the subscription only after validating replacement access; the agent does not remove it.
- **Keep** Reader + Monitoring Contributor.
- **Propose** custom role `<ORG>-SREAgent-VMOperator` on `<RESOURCE_GROUP>` (see [example-vm-operator.json](example-vm-operator.json)); a person creates and assigns it after review.

Risk: the custom role has no wildcards, no `Microsoft.Authorization/*`, no data actions; assignable only to `<RESOURCE_GROUP>`.
Optional additions only if mitigation runbooks need them: `virtualMachines/restart/action`, `virtualMachines/deallocate/action`.
Not recommended: `Microsoft.Authorization/roleDefinitions/write` for the agent; let a human create roles.

## Lessons recorded in SKILL.md
- MI caller in the Activity Log = **object ID**.
- With `--caller`, 30-day windows timed out (exit 124); **10-day slices** worked.
