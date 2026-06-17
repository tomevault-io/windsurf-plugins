---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All scripts run from the repo root unless noted.

- `bun install` — install workspace deps (postinstall patches `@openrouter/agent`)
- `bun test` — runs `@noetic-tools/types`, `@noetic-tools/memory`, `@noetic-tools/core`, `@noetic-tools/code-agent`, `@noetic/eval` suites **sequentially** (does NOT include `@noetic-tools/cli`)
- `bun test:ci` — same plus coverage enforcement (diff gate from baseline)
- `bun run lint` / `bun run lint:fix` — biome
- `bun scripts/check-export-tags.ts` — validates `@public` JSDoc tags on core's entry points
- `cd packages/core && bun run gen:schema` — regenerate the published JSON Schema for dynamic workflows from `WorkflowDocumentSchema`. **MUST run (same commit) whenever you change the JSON-workflow Zod schema in `packages/core/src/schemas/workflow.ts`** — it rewrites both the package artifact (`packages/core/schema/noetic-workflow.schema.json`) and the hosted copy (`packages/web/public/schema/noetic-workflow.schema.json`, served at the schema's `$id`). A drift-gate test fails CI if either is stale. Never hand-edit the generated `*.schema.json` files. See `.claude/rules/sync-spec-code-docs.md` Requirement 6.

In-workspace consumers resolve `@noetic-tools/types`, `@noetic-tools/memory`, and `@noetic-tools/core` straight to `src/*.ts` via the `bun` export condition (tsc matches it through `customConditions`), so no build step is needed for tests, typecheck, or the CLI — a stale `dist/` cannot break the workspace. Only `packages/web` (webpack) still consumes `dist/`; run `bun run build` in those three packages before building web. Published tarballs strip the `bun` conditions via `scripts/strip-dev-conditions.ts` in `prepublishOnly`.

Per-package (`cd packages/<name>`):
- `bun test` — package test suite
- `bun test <path/to/file.test.ts>` — single file
- `bun run typecheck` — `tsc --noEmit`
- `cd packages/cli && bun run dev` — run the CLI from source
- `cd packages/cli && bun run test:e2e` — TUI end-to-end tests (separate from unit tests)

Architecture gates:
- `sentrux check .` — validate `.sentrux/rules.toml` layer + boundary rules
- `sentrux gate .` — quality regression check against the committed `.sentrux/baseline.json`. Runs automatically via `SessionStart` + `Stop` hooks; invoke manually to debug.

## Architecture big picture

Workspace packages under `packages/*`. Dependency direction (arrows = "depends on"):

```
plugins ──→ cli ──→ code-agent ──→ core ←── eval
                                       │
                                       └──→ memory ──→ types ←── sub-harness ←── sub-harness-{claude-code,codex,opencode,pi}
                                                 └───────────↗
web (standalone — no workspace deps)
```

- **`@noetic-tools/types`** — the dependency-free foundation: the conversation `Item` data model, LLM config (`LlmProviderConfig`, `ModelParams`, `LLMResponse`), execution context + steering contracts, the `MemoryLayer` contract (also exported at the `./contract` subpath), platform adapter interfaces, the error model, and the `Item` schema. Imported by `memory` and `core`; depends on nothing in the workspace.
- **`@noetic-tools/memory`** — the memory layer system: lifecycle, budget/projection machinery (`assembleView`, `allocateBudgets`, layer state stores, scoping), and the built-in layers (working/history/observational/plan/temporal/steering/file-reference/static-content/durable-task-state/tool). Depends only on `@noetic-tools/types`; re-exports the `MemoryLayer` contract so it is the one-stop import for memory-layer authoring. Must stay free of imports from `core` (acyclic + tree-shakable).
- **`@noetic-tools/core`** — step primitives (`Step<I,O>` discriminated union), interpreter, runtime, error model, observability. Re-exports the public surface of `@noetic-tools/memory` and `@noetic-tools/types`, so its `.`, `/portable`, `/unstable`, and `/internal/test` entry points are unchanged for consumers. Internal order (foundational → consumer): `types/schemas/errors` → `observability` → `builders/conditions/until` → `runtime` → `interpreter` → `adapters` → `patterns`.
- **`@noetic-tools/sub-harness`** — base contract + helpers for coding-agent sub-harnesses (Claude Code, Codex, opencode, pi). Re-exports the `SubHarness` contract from `@noetic-tools/types` and adds `defineSubHarness`, the turn accumulator, registry, item builders, and error types. Depends only on `@noetic-tools/types`. **`@noetic-tools/core` must never import this package or any `sub-harness-*` adapter** — it resolves adapters from the types contract + a runtime registry, so no agent SDK enters core's dependency graph (enforced by `.sentrux/rules.toml`). See `specs/27-sub-harness-steps.md`.
- **`@noetic-tools/sub-harness-{claude-code,codex,opencode,pi}`** — one adapter per coding agent; each exports a factory (`claudeCode()` etc.) returning a `SubHarness`, backed by the vendor SDK as an optional peer dependency. Used via `step.claudeCode(...)` or a `claude-code` JSON workflow node.
- **`@noetic-tools/code-agent`** — tool implementations, plugin registry, skills, tasks, LSP, git worktree integration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattapperson/noetic](https://github.com/mattapperson/noetic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
