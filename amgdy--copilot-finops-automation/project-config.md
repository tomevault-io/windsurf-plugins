---
trigger: always_on
description: This repository automates GitHub Enterprise Copilot FinOps workflows. Treat changes as governance-sensitive because they can affect billing budgets, cost center membership, and workflow permissions.
---

# Agent Guidance

This repository automates GitHub Enterprise Copilot FinOps workflows. Treat changes as governance-sensitive because they can affect billing budgets, cost center membership, and workflow permissions.

## Core Principles

- Keep config-as-code as the primary production path.
- Keep issue-based config as an optional workflow input path for request/test scenarios or explicitly approved live runs.
- Keep manual mutating workflow runs dry-run by default; scheduled sync/apply runs live only from reviewed file-based config.
- Preserve local/private config safety: files matching `config/*.local.yml` must remain ignored.
- Do not commit tokens, generated reports, logs, JSONL summaries, private enterprise names, user logins, or private cost center data unless the user explicitly asks and confirms they are safe to publish.
- Do not use deprecated request-based Copilot billing terminology. Use AI-credit terminology.

## Requirement Changes Must Update The Skill

Whenever requirements change for any of these areas, update the Copilot skill in `.github/skills/copilot-finops-config/` in the same change:

- config file names or structure
- config schema files (`schemas/`) or the config `version` field
- budget policy fields, defaults, or supported policy types
- cost center member sync fields or behavior
- workflow inputs or issue-form behavior
- issue labels or issue template fields
- validation commands or local run commands
- public/private data safety guidance
- billing terminology or API behavior

At minimum, check and update:

- `.github/skills/copilot-finops-config/SKILL.md`
- `.github/skills/copilot-finops-config/references/interview.md`
- `.github/skills/copilot-finops-config/references/budget-policies.md`
- `.github/skills/copilot-finops-config/references/cost-center-members.md`
- `.github/skills/copilot-finops-config/references/issue-config.md`
- `.github/skills/copilot-finops-config/references/validation.md`

## Config Rules

- `config/copilot-finops.yml` is the **default tracked config** (v2 merged budgets + member mappings) and the per-type workflow default since Phase 3. `config/copilot-finops.example.yml` is its public-safe example. A v2 file declares `version: 2`; `ai_credit_spend_policies` and `team_cost_center_mappings` are both optional (write only what you need).
- `config/budget-policies.yml` and `config/cost-center-members.yml` are the **deprecated** v1 split files. They are frozen but still validated (`schemas/v1/`) and still accepted by the per-type workflows' `*_config_file` inputs (which emit a deprecation notice). Migrate with `scripts/migrate-v1-to-v2.sh`. Do not remove them until the v1 sunset (see `docs/public-release.md`).
- `config/copilot-finops.example.yml` is the public-safe worked example (covers every scope, org/enterprise teams, and both mapping modes) and should stay approachable. The v1 `.example.yml` files have been removed; v1 is exercised via the tracked `config/budget-policies.yml` / `config/cost-center-members.yml` and the schema tests.
- v2 renames the vocab (`budget_policies` -> `ai_credit_spend_policies`, `type` -> `scope`, `type: universal` -> `scope: all_users`, `coverage` -> `credit_scope` with `total_spend`/`additional_spend` -> `pool_then_metered`/`metered_only`; flattened `amount`/`stop_at_limit`/`alert_admins`) and infers enterprise-vs-org from whether `organization:` is set. v1 stays frozen and fully supported; do not mix v1 and v2 vocab in one file. Phase 3 flipped the tracked defaults to the merged v2 file.
- `scope: organization` is dual-track like `team` (requires `organization` + `credit_scope`; forbids `teams`/`cost_center`/`enterprise`/`remove_extra_members`/`users`). `credit_scope: pool_then_metered` -> one user budget per org member (hard-stop); `credit_scope: metered_only` -> one org-scope budget. The entry's parent picks the write endpoint: an `organization:` parent (only `scope: organization`) writes on the org billing endpoint (`/organizations/{org}/settings/billing/budgets`); everything else uses the enterprise endpoint. Cost-center budgets always stay on the enterprise endpoint (the org budgets API has no `cost_center` scope).
- `scope: user` is the multi-user budget: requires `users` (a non-empty list of logins), produces one `budget_scope: user` budget per login on the enterprise endpoint, is always hard-stop (forced `stop_at_limit: true`), and forbids `teams`/`cost_center`/`credit_scope`/`organization`/`remove_extra_members`. Each login takes part in conflict detection, so a later team/org `pool_then_metered` policy on the same login wins (last-wins).
- `scope: team` takes a non-empty `teams` list and is applied to each listed team: `pool_then_metered` unions the teams' members (deduped) into per-member hard-stop user budgets; `metered_only` fans out one derived/auto-created cost-center budget per team. An explicit `cost_center:` is only allowed when exactly one team is listed (it is forbidden with multiple teams, in both the schema and `validate-config.sh`). v1 carries a single `source.team_slug`, which the scripts normalize to a one-element teams list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amgdy/copilot-finops-automation](https://github.com/amgdy/copilot-finops-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
