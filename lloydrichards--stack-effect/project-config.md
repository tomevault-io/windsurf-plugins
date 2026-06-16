---
trigger: always_on
description: > Note: This file is the authoritative source for coding agent instructions. If
---

# AGENTS.md

> Note: This file is the authoritative source for coding agent instructions. If
> in doubt, prefer AGENTS.md over README.md. See nested AGENTS.md files in each
> workspace for app-specific patterns.

## Commands

| Command                           | Purpose                                    |
| --------------------------------- | ------------------------------------------ |
| `bun install`                     | Install dependencies                       |
| `bun dev`                         | Run workspace dev tasks via Turbo          |
| `bun dev --filter=stack-effect`   | Start CLI app in watch mode                |
| `bun run build`                   | Build all workspaces                       |
| `bun lint`                        | Lint with Biome                            |
| `bun format`                      | Format with Biome                          |
| `bun format:check`                | Validate formatting without writing        |
| `bun run type-check`              | Run TypeScript checks across workspaces    |
| `bun run test`                    | Run workspace tests through Turbo + Vitest |
| `bun run test --filter=<package>` | Run tests for a specific workspace         |

## Tech Stack

Bun 1.2+, TypeScript 5.9, Effect 4-beta, Vitest 4, Biome 2.4

## Task Completion Requirements

- All of `bun format`, `bun lint`, and `bun run type-check` must pass before considering a task complete.
- NEVER use `bun test` in this repository. Always use `bun run test` so Turbo workspace filters and task wiring are respected.
- For behavior changes, run a scoped test command (`bun run test --filter=<workspace>`) for impacted workspaces before completion.

## CLI Automation Smoke Test

When validating non-interactive CLI behavior (LLM/CI paths), run commands against a temporary repository and always clean it up.

```bash
TMP_REPO="$(mktemp -d)"
trap 'rm -rf "$TMP_REPO"' EXIT

# 1) Non-interactive init requires project name with --yes
bun run start -- init smoke-app --yes --root "$TMP_REPO"

# 2) Non-interactive add with explicit Selection inputs
bun run start -- add --yes --root "$TMP_REPO" --target package/domain --modules domain-api --dry-run

# 3) Optional negative test: cross-target implication should fail in non-interactive mode
bun run start -- add --yes --root "$TMP_REPO" --target client-react/web --modules http-api-react-client --dry-run
```

Notes:

- Use `trap ... EXIT` so cleanup runs even if a command fails.
- Do not point smoke tests at the current workspace root; always use `mktemp -d`.
- The temporary repo must be removed after the test run.

## Project Snapshot

`stack-effect` is a scaffolding CLI for full-stack TypeScript apps built on Effect. The core flow is:

```text
Selection ──> Blueprint ┬─> Plan ──> Apply ──> ApplyResult
                        ╰───────────────────────────────────> FinalizeReport

```

The CLI (`apps/cli`) orchestrates this flow using shared packages in `packages/*`. Users run `stack-effect init` to create a project and `stack-effect add` to incrementally add targets (client, server, cli, package) with modules (features).

## Core Priorities

1. Deterministic behavior: identical inputs should produce identical blueprint and plan outputs.
2. Correctness over convenience: prefer explicit failures and conflict surfaces over implicit fallbacks.
3. Predictable planning and apply semantics: preserve the Selection/Blueprint/Plan/Apply boundaries.

## Maintainability

- Long-term maintainability is a core priority; extract reusable logic into shared packages instead of duplicating across CLI/services.
- Keep domain contracts in `@repo/domain` as the canonical boundary, then implement runtime behavior in `@repo/scaffold` and `@repo/catalog`.
- Favor small, composable Effect services/layers over one-off local logic.

## Package Roles

- `apps/cli`: Effect CLI entrypoint and command UX (`init`, `add`, `graph`) that composes scaffold services.
- `packages/domain`: Shared Effect Schema contracts and domain vocabulary for Catalog/Selection/Blueprint/Plan/Apply/Finalize.
- `packages/catalog`: Read-only catalog definitions and lookup service for targets/modules and dependency metadata.
- `packages/scaffold`: Runtime orchestration services (blueprint resolution, planning, apply, finalize, formatting).
- `packages/observability`: Shared OpenTelemetry layer wiring for Effect apps.
- `packages/config-typescript`: Shared TypeScript configuration package.

## Hard Domain Rules

- Do not collapse terms: `Selection` (user intent), `Blueprint` (dependency closure), `Plan` (repo-aware outcomes), and `Apply` (execution intent) are distinct.
- `Plan` is policy-free and must not contain apply decisions.
- `ApplyDecision` entries are only for conflicted planned paths; missing or extra decisions are invalid.
- A module contributes only to its owning target; cross-target effects must be modeled via target/module dependencies.
- Preserve canonical domain terminology from `.docs/ubiquitous-language.md` and `.docs/domain-lexicon.md` in code and docs.

## Code Style

- **Formatting**: Spaces (not tabs), double quotes for strings
- **Imports**: Use `@repo/domain` for shared types; Biome auto-organizes imports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lloydrichards/stack-effect](https://github.com/lloydrichards/stack-effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
