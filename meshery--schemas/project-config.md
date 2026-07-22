---
trigger: always_on
description: This is the central schema repository for the Meshery platform. Schemas here drive Go struct generation, TypeScript type generation, and RTK Query client generation. Mistakes in schema design propagate into generated code across multiple downstream repos (meshery/meshery, layer5io/meshery-cloud).
---

# Meshery Schemas - Coding Agent Instructions

This is the central schema repository for the Meshery platform. Schemas here drive Go struct generation, TypeScript type generation, and RTK Query client generation. Mistakes in schema design propagate into generated code across multiple downstream repos (meshery/meshery, layer5io/meshery-cloud).

> **This file is a routing and policy overview.** The critical rules - the
> dual-schema pattern, casing, and generated-artifact warnings - live inline
> below. Long-form reference material lives in `docs/` and is linked from the
> [Detailed references](#detailed-references) section. Keep this file concise:
> Claude Code warns when it exceeds 40k characters.
>
> `CLAUDE.md` is a symlink to this file. Edit `AGENTS.md` only.

## Source of Truth

The source of truth for a construct's API contract depends on where it is in the migration lifecycle:

1. **Pre-migration** - While a construct is being migrated from a downstream repository (e.g., `layer5io/meshery-cloud`) into meshery/schemas, the downstream implementation is the reference for field discovery: field names, types, JSON tags, and database column mappings.
2. **Post-migration** - Once a construct has been fully migrated and its schema is defined here, **meshery/schemas becomes the permanent, authoritative source of truth.** Downstream repositories must conform to the contract defined here, not the reverse.

For constructs that have been migrated:

- When a downstream repository's implementation diverges from the schema contract defined here, **this repository is correct** and the downstream code must be updated.
- When cross-construct consistency requires a change that conflicts with current downstream implementation, make the breaking change here and open issues in affected repositories documenting the required migration.
- Do not weaken schema contracts, skip validation rules, or introduce inconsistent patterns to accommodate legacy downstream code.

## Build

```bash
make build       # generate Go structs + TypeScript types + RTK clients
make validate-schemas  # run repository schema validation rules
npm run build    # build TypeScript distribution (dist/)
```

Generated artifacts (`models/`, `typescript/generated/`) are committed by automation on `master`. The TypeScript distribution in `dist/` is produced by the npm build/publish workflow and is not committed to this repo. Do not edit generated artifacts by hand, and do not manually commit regenerated output in normal PRs unless the change explicitly requires it.

**Never hand-edit generated output:**

- ❌ Generated Go code in `models/`
- ❌ Generated TypeScript in `typescript/generated/`
- ❌ Built files in `dist/`

## Required on Every PR

- **Tests accompany every behavioral change.** Run every locally-runnable test
  before requesting review; never defer runnable coverage to reviewers or
  follow-up PRs.
- **Documentation accompanies every behavioral change, in both forms:**
  - External, user-facing: docs.meshery.io and the published OpenAPI documentation - update whenever the change is
    user-visible.
  - Internal, developer-facing: this repo's [`docs/`](docs/) - update whenever
    architecture, workflows, or contracts change.
- **Schema-aware changes**: run `make validate-schemas && make consumer-audit` before pushing.
- **Sign off every commit** (`git commit -s`).
- **No AI attribution** in commits, PR descriptions, comments, or code.

Always run `make build`, `go test ./...`, and `npm run build` before committing schema changes. See [Common Mistakes](docs/schema-review-checklist.md#common-mistakes-to-avoid) and the [Checklist for Schema Changes](docs/schema-review-checklist.md#checklist-for-schema-changes) for the full pre-PR list.

## AXI Agent Tooling

- Use the `gh-axi` CLI tool to interact with GitHub. Prefer `gh-axi` over `gh`.
- Use `chrome-devtools-axi` for browser automation (navigate, snapshot, click, fill forms, run JS, inspect console/network) in place of raw Playwright/chrome-devtools MCP for ad hoc tasks.
- Run `quota-axi` to check local agent-provider quota windows before long-running work.
- Use the `lavish` skill (`lavish-axi` CLI) to turn a plan, comparison, or report into a reviewable HTML artifact.

## The Dual-Schema Pattern (REQUIRED)

This is the most critical design rule in this repo. Every agent or contributor MUST follow it.

### Rule 1: `<construct>.yaml` = response schema only

The YAML file for an entity represents the **full server-side object** as returned in API responses. It is NOT a request body schema.

**Required properties of every entity `.yaml`:**

- `additionalProperties: false` at the top level
- All server-generated fields defined in `properties`: `id`, `created_at`, `updated_at`, `deleted_at`
- Server-generated fields that are always present belong in `required`

### Rule 2: Every writable entity needs a `*Payload` schema in `api.yml`

For any entity that has `POST` or `PUT` operations, define a `{Construct}Payload` schema in `api.yml` that:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meshery/schemas](https://github.com/meshery/schemas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
