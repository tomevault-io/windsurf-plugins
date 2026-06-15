---
trigger: always_on
description: > VS Code Copilot-specific orchestration instructions.
---

# APEX - Copilot Instructions

> VS Code Copilot-specific orchestration instructions.
> For general project conventions, build commands, and code style, see the root `AGENTS.md`.

## Azure Defaults (canonical)

This section is the canonical declaration of Azure infrastructure defaults.
Every skill, agent, and prompt must reference this section — never restate
the values inline. The IaC-flavoured mirror with CAF naming, AVM modules,
and reference index lives in
[`.github/skills/azure-defaults/SKILL.md`](skills/azure-defaults/SKILL.md).

### Default Regions

| Service             | Default Region       | Reason                         |
| ------------------- | -------------------- | ------------------------------ |
| **All resources**   | `swedencentral`      | EU GDPR-compliant              |
| **Static Web Apps** | `westeurope`         | Not available in swedencentral |
| **Failover**        | `germanywestcentral` | EU paired alternative          |

### Required Tags (Azure Policy Enforced)

Tag schema is **whatever live Azure Policy enforces** in the target
subscription. Governance Discovery (Step 3.5) discovers the real
contract via `discover.py` and writes it to
`04-governance-constraints.json` (`tag_contract.tags[]`,
`tag_contract.source: "policy"`); that always wins.

**Greenfield fallback** (no tag policy found at any inherited scope):
the APEX-standard 9-tag set — `environment`, `owner`, `costcenter`,
`application`, `workload`, `sla`, `backup-policy`, `maint-window`,
`technical-contact` — all lowercase. This mirrors the org-wide
resource-group tag-deny policy (every key must exist on the RG or the
deployment is denied). Citation + greenfield decision checklist:
[`azure-defaults/references/tag-strategy.md`](skills/azure-defaults/references/tag-strategy.md).

> The PascalCase set (`Environment`, `ManagedBy`, `Project`, `Owner`)
> is a **deprecated convention** retained only for backward
> compatibility on existing projects whose deployed resources already
> carry that casing. Do not propagate it to new projects. `ManagedBy`
> and `Project` are not part of the required contract — `ManagedBy` may
> still be emitted as an optional deploy-provenance marker.

### Security baseline + AVM mandate

Non-negotiable: HTTPS-only, TLS 1.2 minimum, no public blob, public network
disabled for prod data services, Managed Identity over keys, AVM-first.
Full rules:
[`iac-policy-compliance.md`](instructions/references/iac-policy-compliance.md)
and
[`iac-security-baseline.md`](instructions/references/iac-security-baseline.md).

### SKU source of truth

Creative SKU decisions (App Service, VM, SQL, Cosmos, AKS pools, Redis,
APIM, App Gateway, Storage replication) flow through
`agent-output/{project}/sku-manifest.{json,md}` — never re-derive SKUs
from artifact prose. Authoring rules:
[`sku-manifest.instructions.md`](instructions/sku-manifest.instructions.md).

## Session State — apex-recall

All session state flows through `apex-recall`. Do not read or write
`00-session-state.json` directly.

```bash
# Lifecycle
apex-recall init <project> --json                                    # new project
apex-recall show <project> --json                                    # context: step, decisions, findings, artifacts
apex-recall checkpoint <project> <step> <phase> --json               # after each phase
apex-recall complete-step <project> <step> --json                    # on step completion
apex-recall review-audit <project> <step> ... --json                 # after challenger reviews

# Atomic step transition — PREFERRED for moving between steps. Bundles
# complete-step (with challenger gate) + decide + start-step into one
# 00-session-state.json write, avoiding partial-update drift.
apex-recall transition <project> --from-step <s> --to-step <t> \
    --complete --decision key=value --json

# Decisions + findings
apex-recall decide <project> --key <k> --value <v> --json
apex-recall decide <project> --decision "<text>" --rationale "<why>" --json
apex-recall finding <project> --add "<text>" --json

# Read-only orientation: sessions | files | search '<term>' | decisions (all accept --json)
```

If `apex-recall` returns useful context, skip redundant file reads.
If empty/errored, continue normally — it's a convenience, not a blocker.

Canonical `show --json` schema (including the `session.steps` shape and
jq query templates) lives at
[`tools/apex-recall/docs/show-schema.md`](../tools/apex-recall/docs/show-schema.md).
The valid decision-keys registry lives at
[`tools/apex-recall/docs/decision-keys.md`](../tools/apex-recall/docs/decision-keys.md).

## Multi-Step Workflow

The Steps 1–7 + Post-Lessons table is in [AGENTS.md](../AGENTS.md#agent-workflow);
the machine-readable source is
[`.github/skills/workflow-engine/templates/workflow-graph.json`](skills/workflow-engine/templates/workflow-graph.json).
Each step's outputs land in `agent-output/{project}/`; context flows via artifact
files + handoffs. Reviews are adversarial passes by challenger subagents —
**default flow is single-pass `comprehensive`** (mandatory at Steps 1, 2, 4;
Step 3.5 uses `governance-reconciliation`). Multi-pass deep review is **opt-in
only** via `decisions.review_depth = "deep"` or an explicit `10-Challenger`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonathan-vella/apex](https://github.com/jonathan-vella/apex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
