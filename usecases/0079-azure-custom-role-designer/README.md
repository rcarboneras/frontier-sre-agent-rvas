# Azure RBAC and Entra custom role designer

Role Designer designs, reviews, right-sizes and audits Azure RBAC and Microsoft Entra ID custom roles. It checks built-in roles first, traces each proposed permission to a task, observed operation or documented dependency, and flags privilege-escalation paths.

Every design answer contains a recommendation and scope, role JSON, an action-to-evidence table, a risk review, deployment commands and next steps. It can create or update an Azure role definition only after approval. Entra changes, role assignments and PIM eligibility are delivered as scripts for a person to run. It never deletes roles or removes assignments.

Adapted from [`rcarboneras/SRE-Agent`](https://github.com/rcarboneras/SRE-Agent), commit [`6bb95263aa3888c79ffa67c293e74b40fa1750dd`](https://github.com/rcarboneras/SRE-Agent/commit/6bb95263aa3888c79ffa67c293e74b40fa1750dd). See [provenance and adaptations](docs/README.md).

## Components

| Component | File | Purpose |
|-----------|------|---------|
| Skill | [azure-custom-role-designer](manifests/skills/azure-custom-role-designer/SKILL.md) | Six-step workflow, Azure and Entra JSON templates, and sanitized worked examples |
| Specialist | [role-designer.yaml](manifests/subagents/role-designer.yaml) | Framework-deployable `role_designer` subagent with three native CLI tools |
| Portal export | [role_designer.agent.yaml](docs/role_designer.agent.yaml) | `AgentConfiguration` for the portal's custom-agent YAML editor |

No Python tool, MCP connector or scheduled task is required. The skill uses native `RunAzCliReadCommands`, `GetAzCliHelp` and `RunAzCliWriteCommands`. Microsoft Graph identity lookups use read-only `az ad` commands.

## Prerequisites

- A deployed Azure SRE Agent and **SRE Agent Administrator** access for the installer and approver.
- **Reader** for the agent's execution identity on the subscriptions or resource groups being analyzed.
- Directory read access when resolving users, groups, apps or service principals. If `az ad` returns 403, have a directory administrator review the required Graph application permissions; upstream uses **Directory.Read.All**. This is tenant-wide access: do not grant it by default or grant Graph write permissions.
- **Microsoft Entra ID P1** for Entra custom roles; the appropriate PIM licensing for eligible assignments.
- For the framework CLI: Bash, `jq`, Python 3 with PyYAML, and `az` and `curl` for deployment.

Keep role creation with a person by default. In that configuration, do not grant the agent `Microsoft.Authorization/roleDefinitions/write`: it delivers JSON and commands instead. If agent-executed creation or updates are explicitly needed, an administrator must grant `roleDefinitions/read` and `roleDefinitions/write` at the intended scope. Treat that write permission as HIGH risk and apply the approval controls below before granting it. Do not grant Owner, role-assignment write or delete permissions for this use case.

To grant only the read access, an authorized administrator can run:

```bash
az role assignment create \
  --assignee-object-id <AGENT_PRINCIPAL_ID> \
  --assignee-principal-type ServicePrincipal \
  --role Reader \
  --scope /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>
```

Use the principal (object) ID of the actual execution identity, not its client ID.

## Setup

### 1. Enforce approval

Before using the specialist, set **Settings > Basics > Review** as the agent's default mode. Under **Settings > Tool access policies > Global**, add an **Ask** rule for `RunAzCliWriteCommands`. In Autonomous mode an Ask rule can be approved automatically, so the prompt alone is not an approval boundary.

The specialist only permits `az role definition create` and `az role definition update` as writes. It must ask before either one. It must not execute assignments, deletion, `az rest` or Graph writes. An administrator approves each proposed change.

### 2. Install through the framework

Run these commands from the repository root, or from the root of the extracted download:

```bash
usecases/_framework/usecases.sh deploy azure-custom-role-designer \
  --endpoint <AGENT_ENDPOINT> --dry-run
usecases/_framework/usecases.sh deploy azure-custom-role-designer \
  --endpoint <AGENT_ENDPOINT>
usecases/_framework/usecases.sh verify azure-custom-role-designer \
  --endpoint <AGENT_ENDPOINT>
```

`deploy` installs the skill, including its templates and examples, before the specialist. It does not configure tool access policies, grant Azure or Graph permissions, or substitute placeholders. No environment-specific values need to be written into the checked-in files: provide the target identity and scope in chat.

Use `usecases.sh diff azure-custom-role-designer --endpoint <AGENT_ENDPOINT>` to compare local manifests with the live agent. Do not paste the framework `SubAgent` manifest into the portal editor; use the portal export below for that route.

### 3. Alternative: install in the portal

1. In **Builder > Create > Skill**, create `azure-custom-role-designer` with the exact name and description from its `SKILL.md` frontmatter. Paste the full skill content, then upload the two JSON templates and the `examples/` folder, preserving their relative paths. Leave the skill's tools empty.
2. In **Builder > Agent Canvas > Create > Custom agent > YAML**, paste [docs/role_designer.agent.yaml](docs/role_designer.agent.yaml).
3. In the Form tab, confirm the name `role_designer`, skill `azure-custom-role-designer` and tools `RunAzCliReadCommands`, `GetAzCliHelp`, `RunAzCliWriteCommands`. Save.
4. Confirm the Review mode and Ask policy from step 1 before testing a write in a normal chat.

The portal export retains upstream's `agent_type: Autonomous` editor value. It is not a substitute for the global Review mode and Ask policy. If the editor rejects the YAML, create the custom agent in the Form tab using the same prompt, skill and tools.

## Run a review

In a new thread, open `/agent` and select `role_designer`. For example:

> Right-size the permissions of `<IDENTITY_OBJECT_ID>` on `<RESOURCE_GROUP_SCOPE>`. Check built-in roles first, compare observed operations with existing assignments, and deliver JSON, evidence and risk review. Do not change anything.

Other supported requests include designing a role from a task description, reviewing supplied JSON, or auditing custom roles in a subscription. For Entra roles or PIM, run the supplied read-only discovery script yourself and return its output before requesting a design.

Run the [test prompts and approval checks](tests/README.md) before allowing production use. Test playground threads are read-only; verify the approval card in a normal thread with a disposable scope.

## Limits

- Activity Log evidence normally covers at most 90 days and does not establish read or data-plane requirements. Query in small slices, disclose failed or truncated slices and check an empty result against a known event. Missing evidence is not proof that a permission is unused.
- `az ad` cannot inspect Entra directory role definitions, assignments or PIM schedules. The agent provides Microsoft Graph PowerShell scripts for the user to run; it does not use `az rest`.
- Creating or updating an Azure role definition does not assign it. Widening `AssignableScopes` does not widen existing assignments. Review assignments separately.
- Examples use placeholders and illustrative evidence. They are not a production permission recommendation or evidence of validation in this repository.

## Maintain and status

Update the skill and both agent representations together. Keep the `system_prompt`, tools and allowed skill aligned between the portal export and the framework manifest; only their envelope and framework descriptions differ.

Status is **built**, not **validated**. Upstream describes portal installation and testing; this adaptation has not been deployed to a live agent and includes no verified sample report. The test checklist is provided for the first deployment.
