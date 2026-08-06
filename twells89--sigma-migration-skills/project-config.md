---
trigger: always_on
description: Migration skills for moving BI tools (**Tableau, Power BI, Qlik, ThoughtSpot,
---

# AGENTS.md — sigma-migration-skills

Migration skills for moving BI tools (**Tableau, Power BI, Qlik, ThoughtSpot,
QuickSight, Looker, Cognos, MicroStrategy**) to
**Sigma**: per-tool *converters* (source → Sigma data model + workbook, with
warehouse parity verification) and read-only *assessments* (tenant inventory →
migration-readiness readout + shortlist).

This repo is packaged as a Claude Code plugin marketplace, but the skills are
agent-neutral: each is a `SKILL.md` (instructions) plus `scripts/` (Ruby/Python/
shell) and `refs/`. Any coding agent (Cursor, Cortex Code, etc.) can run them by
reading the relevant `SKILL.md` and executing its scripts.

## How to use a skill

1. Pick the skill from the index below that matches the user's intent.
2. **Read that skill's `SKILL.md` in full first** — it is the source of truth for
   the phased workflow. The `refs/*.md` next to it cover details.
3. Run its scripts **from the skill directory** (script paths are relative, e.g.
   `scripts/setup.rb`). `cd` into the skill dir, then invoke.

## Skill index

| Intent | Skill | Path (read its `SKILL.md`) |
|---|---|---|
| Convert a Tableau datasource/workbook → Sigma | `tableau-to-sigma` | `plugins/tableau-to-sigma/skills/tableau-to-sigma/` |
| Scope/assess a Tableau site for migration | `tableau-assessment` | `plugins/tableau-to-sigma/skills/tableau-assessment/` |
| Convert a Power BI report + semantic model → Sigma (DAX translation) | `powerbi-to-sigma` | `plugins/powerbi-to-sigma/skills/powerbi-to-sigma/` |
| Scope/assess a Power BI / Fabric tenant | `powerbi-assessment` | `plugins/powerbi-to-sigma/skills/powerbi-assessment/` |
| Convert a Qlik Sense / Qlik Cloud app → Sigma | `qlik-to-sigma` | `plugins/qlik-to-sigma/skills/qlik-to-sigma/` |
| Scope/assess a Qlik Cloud tenant | `qlik-assessment` | `plugins/qlik-to-sigma/skills/qlik-assessment/` |
| Convert a ThoughtSpot model + Liveboards → Sigma (TML) | `thoughtspot-to-sigma` | `plugins/thoughtspot-to-sigma/skills/thoughtspot-to-sigma/` |
| Scope/assess a ThoughtSpot instance | `thoughtspot-assessment` | `plugins/thoughtspot-to-sigma/skills/thoughtspot-assessment/` |
| Convert an Amazon QuickSight analysis/dashboard → Sigma | `quicksight-to-sigma` | `plugins/quicksight-to-sigma/skills/quicksight-to-sigma/` |
| Scope/assess a QuickSight instance | `quicksight-assessment` | `plugins/quicksight-to-sigma/skills/quicksight-assessment/` |
| Convert a Looker (LookML model + dashboards) → Sigma | `looker-to-sigma` | `plugins/looker-to-sigma/skills/looker-to-sigma/` |
| Scope/assess a Looker instance | `looker-assessment` | `plugins/looker-to-sigma/skills/looker-assessment/` |
| Convert an IBM Cognos data module + report → Sigma | `cognos-to-sigma` | `plugins/cognos-to-sigma/skills/cognos-to-sigma/` |
| Scope/assess a Cognos Analytics instance | `cognos-assessment` | `plugins/cognos-to-sigma/skills/cognos-assessment/` |
| Convert a MicroStrategy dossier + semantic model → Sigma | `microstrategy-to-sigma` | `plugins/microstrategy-to-sigma/skills/microstrategy-to-sigma/` |
| Scope/assess a MicroStrategy (Strategy One) instance | `microstrategy-assessment` | `plugins/microstrategy-to-sigma/skills/microstrategy-assessment/` |
| Convert a Sisense (ElastiCube / Live model + dashboards) → Sigma | `sisense-to-sigma` | `plugins/sisense-to-sigma/skills/sisense-to-sigma/` |
| Scope/assess a Sisense instance | `sisense-assessment` | `plugins/sisense-to-sigma/skills/sisense-assessment/` |
| Convert a GoodData Cloud / .CN workspace (LDM + MAQL + insights + dashboards) → Sigma | `gooddata-to-sigma` | `plugins/gooddata-to-sigma/skills/gooddata-to-sigma/` |
| Scope/assess a GoodData workspace | `gooddata-assessment` | `plugins/gooddata-to-sigma/skills/gooddata-assessment/` |
| Convert a Domo dashboard (DataSets + Beast Modes + cards) → Sigma | `domo-to-sigma` | `plugins/domo-to-sigma/skills/domo-to-sigma/` |
| Scope/assess a Domo instance | `domo-assessment` | `plugins/domo-to-sigma/skills/domo-assessment/` |
| Land a Tableau published-datasource/extract in Snowflake or Databricks | `tableau-vds-to-cdw` | `plugins/tableau-to-sigma/skills/tableau-vds-to-cdw/` |
| Land an Import-mode Power BI model's data in Snowflake (before converting) | `powerbi-import-to-snowflake` | `plugins/powerbi-to-sigma/skills/powerbi-import-to-snowflake/` |

Assessments are read-only (never write to the source or post to Sigma); run one
to pick what to convert, then hand off to the matching converter.

Each converter's phase numbering is local to its SKILL.md — the canonical
Assess → Discover → Reuse-check → Convert → Post-DM gate → Build workbook →
Layout → Parity → Security → Enhance arc, with the per-skill phase-number
mapping, is in [`docs/phase-schema.md`](docs/phase-schema.md). Never renumber
a skill's phases.

## Governance (keep skills consistent)

See [`CONTRIBUTING.md`](CONTRIBUTING.md). Enforced by CI:

- **Shared infra** lives in [`shared/`](shared/) (single source of truth) and is
  vendored byte-identical into each plugin. Edit the canonical copy, then
  `ruby tools/sync-shared.rb`. `tools/check-shared.rb` fails CI on drift.
- **Mandatory arc gates** (reuse / readback / layout-last / parity / RLS) are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twells89/sigma-migration-skills](https://github.com/twells89/sigma-migration-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
