---
trigger: always_on
description: **EventCatalog Agents** are [Flue](https://flueframework.com)-powered AI agents that keep
---

# Repository Guidelines

**EventCatalog Agents** are [Flue](https://flueframework.com)-powered AI agents that keep
EventCatalog documentation in sync with your code. The product is the agent; today it is delivered as
a composite **GitHub Action** that runs in CI, but CI is a delivery surface, not the limit of the
agent. Keep this distinction in mind when naming things and writing docs: reach for "the agent" for
behavior, and "the action" only for GitHub Action mechanics (inputs, checkout, `$GITHUB_ACTION_PATH`,
permissions).

There are two agents today. Each Action invocation runs exactly one, selected by the `agent` input.

**Code-to-Docs** (`agent: code-to-docs`, workflow `pr-review`) is a PR reviewer that:

- reads source pull request diffs,
- understands services, events, commands, queries, schemas, channels, containers, and domain changes,
- inspects the checked-out EventCatalog catalog,
- plans which catalog resources should change, then updates the matching documentation,
- opens or updates a pull request in the catalog repository,
- comments back on the source pull request with a concise summary and catalog PR link.

**Breaking Changes** (`agent: breaking-changes`, workflow `breaking-changes`) is a read-only PR
reviewer that:

- finds changed message schema files in the diff (`.json`, `.yml`, Avro, Protobuf, GraphQL, etc.),
- scores each schema change for whether it is breaking for existing consumers,
- traces breaking changes through the catalog to the resources that consume the message,
- comments back on the source pull request with the breaking lines and affected consumers.
- It never edits the catalog.

Keep deterministic behavior in TypeScript. Let the agents reason about documentation and breakage,
but keep GitHub writes, path resolution, diff parsing, token preflight, impact-plan enforcement, and
catalog change detection explicit in code.

## Project Structure

- `action.yml` — composite GitHub Action. Sets up pnpm and Node 24, checks out the configured catalog
  repo into `eventcatalog/`, installs dependencies into `$GITHUB_ACTION_PATH`, then maps the `agent`
  input to its workflow (`code-to-docs` → `pr-review`, `breaking-changes` → `breaking-changes`) and
  runs `pnpm exec flue run <workflow> --root "$GITHUB_ACTION_PATH" --target node`.
- `flue.config.ts` — Flue config for the Node target.
- `src/workflows/pr-review.ts` — the Code-to-Docs workflow orchestrating PR review and catalog update.
- `src/workflows/breaking-changes.ts` — the Breaking Changes workflow: detect schema changes, score
  them, trace consumers, and comment back. Read-only.
- `src/agents/code-to-docs.ts` — Flue agent: model selection, sandbox setup, EventCatalog skill
  registration, and `dump_catalog` / `linter` tool wiring.
- `src/agents/breaking-changes.ts` — read-only Flue agent for the Breaking Changes workflow
  (`dump_catalog` only, no linter; contract-focused instructions).
- `src/prompts/` — the agent prompts run by the workflows:
  - `create-documentation-plan-from-code-changes.ts` — Code-to-Docs read-only impact-planning pass.
  - `apply-documentation-plan-to-catalog.ts` — Code-to-Docs: applies the approved plan to the catalog.
  - `detect-breaking-schema-changes.ts` — Breaking Changes: scores one schema diff.
  - `find-schema-consumers.ts` — Breaking Changes: traces a breaking schema to its catalog consumers.
- `src/skills/eventcatalog-documentation/` — local Agent Skill (`SKILL.md` + `references/`) telling
  the agent how to generate EventCatalog resources correctly.
- `src/tools/dump-catalog.ts` — deterministic tool returning the EventCatalog SDK catalog dump.
- `src/tools/linter.ts` — deterministic tool that lints the catalog after changes.
- `src/utils/eventcatalog-utils.ts` — catalog path resolution and catalog checkout inspection.
- `src/utils/diff.ts` — changed-file discovery and ignore-path filtering.
- `src/utils/schema-detection.ts` — pure helper: which changed files are message schemas (Breaking
  Changes).
- `src/utils/impact-plan.ts` — detects catalog changes made outside the approved impact plan.
- `src/utils/github/catalog-pr.ts` — catalog repo preflight, git commit/push, and catalog PR
  create/update logic.
- `src/utils/github/reporter.ts` — source PR comment create/update logic (one self-updating comment
  per agent: Code-to-Docs summary and Breaking Changes report use separate markers).
- `src/utils/analytics.ts` — anonymous PostHog usage analytics (one event per run, per agent).
- `src/review-output.ts` — Valibot schemas for the structured agent responses (impact plan + apply
  result; breaking-change score + schema consumers).
- `evals/` — the eval suite that runs the real agent against fixtures. See `evals/README.md`.
- `README.md` — user-facing action usage and setup notes.

## Runtime Flow

### Code-to-Docs (`pr-review`)

The `pr-review` workflow does the following:

1. Resolve config from the Flue payload and GitHub Action environment variables.
2. Inspect the checked-out catalog directory and exit early with a source PR comment if it is missing
   or invalid.
3. Preflight the catalog repository and target branch with the catalog token before doing model work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [event-catalog/agents](https://github.com/event-catalog/agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
