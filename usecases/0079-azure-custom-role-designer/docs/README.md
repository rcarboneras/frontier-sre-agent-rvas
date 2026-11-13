# Provenance and adaptations

Source: [rcarboneras/SRE-Agent](https://github.com/rcarboneras/SRE-Agent), revision `6bb95263aa3888c79ffa67c293e74b40fa1750dd`.

| Source path | Package path |
|-------------|--------------|
| `Agents/README.md` | Adapted installation walkthrough in `../README.md` |
| `Agents/role_designer.agent.yaml` | `role_designer.agent.yaml` (portal) and `../manifests/subagents/role-designer.yaml` (framework) |
| `Skills/azure-custom-role-designer/` | `../manifests/skills/azure-custom-role-designer/` |

The framework expects `kind: SubAgent` with `metadata.name`; the portal export uses `kind: AgentConfiguration` with `spec.name`. The system prompt, native tool list and allowed skill are the same in both. The framework manifest adds a description and handoff description.

The worked example has been generalized to remove source-environment subscription and identity IDs, resource names and ownership details. Its role file is named `example-vm-operator.json`. The skill's approval wording explicitly depends on Review mode and an Ask tool access policy; the built-in-role comparison must not accept sensitive extra permissions just because their count is small.

No runtime screenshots, sample report or live validation claim has been imported. The source revision has no standalone license file; this is a contribution of the source owner's supplied content under the destination repository's contribution process.
