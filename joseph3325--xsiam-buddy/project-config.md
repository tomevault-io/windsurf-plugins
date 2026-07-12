---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code **plugin** that helps security engineers develop Cortex XSIAM/XSOAR content. The plugin bundles skills (content generation workflows) and reference knowledge files. There is no runtime code — no build step, no tests, no package manager.

## Plugin Architecture

The plugin is installed via `claude plugin` and exposes skills to Claude Code. Skills are Markdown files with YAML front-matter (`SKILL.md`) that Claude reads to guide content generation. Each skill reads its own reference files before generating output.

```
.claude-plugin/
  plugin.json          # Plugin metadata (name, version, author)
  marketplace.json     # Marketplace manifest listing skills
skills/
  xsiam-scripts/       # Standalone Python scripts embedded in YAML
    SKILL.md
    references/script-yaml-spec.md
    references/script-types-patterns.md
  xsiam-integrations/  # Multi-command integrations with BaseClient
    SKILL.md
    references/integration-yaml-spec.md
    references/integration-patterns.md
  xsiam-event-collectors/ # Event collector integrations (data lake ingestion)
    SKILL.md
    references/event-collector-spec.md
  xsiam-xql/           # Standalone XQL query generation
    SKILL.md
    references/xql-examples.md
  xsiam-widgets/      # Dashboard widget queries with | view graph
    SKILL.md
    references/widget-view-graph-spec.md
  xsiam-correlations/  # Correlation rule JSON files with embedded XQL
    SKILL.md
    references/correlation-rule-spec.md
    references/correlation-examples.md
  xsiam-splunk-to-xql/ # SPL to XQL translation
    SKILL.md
    references/spl-to-xql-mapping.md
  xsiam-playbooks/     # Playbook YAML + companion docs
    SKILL.md
    references/playbook-format.md
  xsiam-docs-playbooks/ # Playbook documentation (HTML for Google Docs)
    SKILL.md
    references/playbook-doc-spec.md
    references/html-styling-guide.md
  xsiam-docs-scripts/  # Script documentation (HTML for Google Docs)
    SKILL.md
    references/script-doc-spec.md
  xsiam-comment/       # Reference files for the /xsiam-comment command (no SKILL.md; not a skill)
    references/commenting-style.md
  xsiam-shared/
    references/
      common-patterns.md          # Shared Python patterns (scripts + integrations)
      xql-core-reference.md       # Shared XQL stages, functions, operators (always loaded)
      xql-datasets-core.md        # Shared XQL datasets, presets, joins (always loaded)
      xql-advanced-functions.md   # Advanced array/JSON/window functions (on-demand)
      xql-datasets-extended.md    # Third-party, email, CIE datasets (on-demand)
      xql-federated-search.md     # External S3/GCS/Azure querying (on-demand)
commands/
  xsiam-comment.md     # /xsiam-comment slash command (comments staged XSIAM content)
  xsiam-validate.md    # /xsiam-validate slash command (validates staged content against import-format rules)
docs/plans/            # Design documents for significant changes
tests/fixtures/        # Test fixtures (e.g., uncommented files for /xsiam-comment)
```

## How Skills Work

Each `SKILL.md` follows this pattern:
1. YAML front-matter with `name` and `description` (used for skill matching)
2. A "Before Starting" section listing which reference files to read first
3. A step-by-step workflow the model follows
4. A validation checklist before delivering output

Skills never generate code outside of YAML/JSON output files. All XSIAM content is delivered as unified `.yml` files (Python embedded inside YAML via `script: |-` for scripts, `script.script: |-` for integrations and event collectors), except correlation rules which are delivered as `.json` files matching the XSIAM export/import format.

The xsiam-xql, xsiam-correlations, xsiam-splunk-to-xql, and xsiam-widgets skills use **tiered reference loading**. Core XQL references always load; specialized references (advanced functions, extended datasets, federated search) load on-demand based on query requirements. This reduces token cost for simple queries while keeping advanced knowledge available.

## Scripts vs Integrations vs Event Collectors

The key structural differences between the three Python skill types:

| | xsiam-scripts | xsiam-integrations | xsiam-event-collectors |
|---|---|---|---|
| Use when | Standalone data processing | Connecting to external APIs | Ingesting events into data lake |
| Python location in YAML | Top-level `script: |-` | Nested `script.script: |-` | Nested `script.script: |-` |
| Base class | None (direct demisto calls) | `BaseClient` subclass | `BaseClient` subclass |
| YAML fetch flag | N/A | `isfetch: true` | `isfetchevents: true` |
| Data output | `return_results()` | `demisto.incidents()` (fetch) | `send_events_to_xsiam()` |
| Data destination | War room / context | Incident queue | XSIAM data lake (XQL queryable) |
| Naming convention | Any | PascalCase | Must end with `EventCollector` |

## XQL Skill Family

Four skills share a common XQL reference layer under `xsiam-shared/references/`:

| Skill | Purpose | Output |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joseph3325/xsiam-buddy](https://github.com/joseph3325/xsiam-buddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
