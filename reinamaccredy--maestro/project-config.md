---
trigger: always_on
description: **Generated:** 2026-04-22 06:13:30 +0700
---

@CLAUDE.md

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-22 06:13:30 +0700
**Commit:** 0e72e7ec
**Branch:** feat/task-contracts-stack

## OVERVIEW
Maestro is a local-first conductor for multi-agent software engineering. It is a single-package Bun/TypeScript CLI with an OpenTUI dashboard, repo-owned agent surfaces, and shared project state under `.maestro/`.

## STRUCTURE
```text
maestro/
├── .factory/    # committed bootstrap/reference assets
├── .maestro/    # repo-tracked project state, plans, tasks, and context
├── hooks/       # session/tool hook entrypoints
├── scripts/     # build, version, install, release, and TUI helpers
├── skills/      # shipped built-in skill sources
├── src/         # feature-first CLI + TUI source tree
└── tests/       # unit, integration, and compiled-binary coverage
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| CLI entry and command registration | `src/index.ts` | Commander root; keeps registration thin |
| Dependency wiring | `src/services.ts` | Composition root only |
| Feature boundaries and imports | `src/features/`, `scripts/check-feature-boundaries-lib.ts` | Cross-feature deep imports are forbidden |
| Mission Control flow | `src/infra/commands/mission-control.command.ts`, `src/tui/README.md`, `src/tui/state/snapshot.ts` | Preview, JSON, and render-check stay read-only |
| Built-in skills | `skills/built-in/`, `scripts/sync-built-in-skills.ts` | `src/infra/domain/built-in-skill-templates.ts` is generated |
| Release and install behavior | `scripts/build.ts`, `scripts/ci.ts`, `scripts/install-local.ts`, `.github/workflows/` | `ci.ts` is local release-prep, not a harmless smoke script |
| Daily task loop vs mission workflow | `.maestro/tasks/tasks.jsonl`, `README.md`, `.maestro/MAESTRO.md` | `task` and `mission` are separate systems |
| Compiled-binary verification | `tests/e2e/`, `tests/helpers/run-compiled-cli.ts` | Distinguish `./dist/maestro` from installed `maestro` |

## CODE STYLE
- Prefer `interface` for object shapes and `type` for unions/intersections.
- Avoid `any`; prefer `unknown` plus narrowing.
- Keep top-level/public functions named and give public APIs explicit return types.
- Prefer `undefined` over `null`; use optional chaining and nullish coalescing.
- Use `describe`/`it`; mock external dependencies, not internal modules.

## CONVENTIONS
- Bun-first, ESM, strict TypeScript. There is no repo-wide lint layer; `typecheck` is advisory in CI.
- `src/` is feature-first: `features/` owns domains, `infra/` owns plumbing, `shared/` owns generic utilities, `tui/` owns Mission Control projection/rendering.
- Keep `src/index.ts` and `src/services.ts` thin. Put behavior in the owning feature or infra use case.
- Cross-feature imports go through `@/features/<name>` public surfaces only.
- `skills/built-in/` is the source of truth for shipped skills. Sync it into generated templates; do not hand-edit the generated embed file.
- When adding a new agent-facing feature or changing related agent behavior, update the install/bootstrap agent instruction block in `src/infra/domain/bootstrap-templates.ts` in the same change so injected `.maestro/AGENTS.md` guidance stays current.
- `buildSnapshot()` and `buildHomeSnapshot()` are read models. Preview, JSON, and render-check paths must remain inspection-only.
- Treat `./dist/maestro` and installed `maestro` on `PATH` as different artifacts. Verify which binary was exercised.
- Repo-tracked behavior changes bump the CLI version. Docs-only/comment-only changes do not.

## ANTI-PATTERNS
- Deep imports into another feature's `commands/`, `usecases/`, `domain/`, `ports/`, or `adapters/`.
- Hidden writes or recovery logic inside Mission Control snapshot/preview paths.
- Hand-editing `src/infra/domain/built-in-skill-templates.ts`.
- Assuming `bun run ci` is a generic verification command; it performs release-prep work and may reset git state.
- Treating `task` and `mission` as interchangeable.
- Assuming installed `maestro` is the fresh build without checking `command -v maestro` and the build/install path used.

## COMMANDS
```bash
bun run build
bun run check:boundaries
bun run check:skills
bun run test
./dist/maestro mission-control --render-check --size 120x40
bun run release:local
```

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **maestro** (8293 symbols, 15357 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ReinaMacCredy/maestro](https://github.com/ReinaMacCredy/maestro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
