---
trigger: always_on
description: This repository is a fork of [OpenCode](https://github.com/anomalyco/opencode). BanyanCode adds: (1) an **orchestrator + subagent mesh** for parallel multi-agent workflows, (2) **cross-session memory** with JSONB payloads, (3) a **tree-sitter code graph** utility, and (4) a **researcher agent** with free web search via DuckDuckGo. BanyanCode is TUI/CLI only — `desktop`, `web`, `app`, `storybook` packages are explicitly out of scope.
---

# BanyanCode

This repository is a fork of [OpenCode](https://github.com/anomalyco/opencode). BanyanCode adds: (1) an **orchestrator + subagent mesh** for parallel multi-agent workflows, (2) **cross-session memory** with JSONB payloads, (3) a **tree-sitter code graph** utility, and (4) a **researcher agent** with free web search via DuckDuckGo. BanyanCode is TUI/CLI only — `desktop`, `web`, `app`, `storybook` packages are explicitly out of scope.

See [`ARCHITECTURE.md`](ARCHITECTURE.md) for repo layout, runtime layers, and the BanyanCode service architecture. Per-feature design lives in `specs/banyancode/`. Active work is tracked via issues and PRs — there is no separate "implementation plan" doc.

## Branch, commit, and PR conventions

- Default branch is `main`; use `main` or `origin/main` for diffs and as the base for PRs. Tags ship from `main` directly.
- Branch names: ≤ three words, hyphen-separated, no type prefixes (`feat/`, `fix/`). Examples: `session-recovery`, `fix-scroll-state`, `regenerate-sdk`.
- Commits and PR titles: `type(scope): summary`. Valid types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`. Useful scopes: `core`, `opencode`, `tui`, `sdk`, `plugin`.
- One logical change per commit. Run `bun typecheck` and the relevant `bun test` between commits.
- Regenerate the JS SDK after any HTTP route or schema change: `./packages/sdk/js/script/build.ts`.

## Style guide

- Keep things in one function unless composable or reusable. Don't extract single-use helpers preemptively.
- Inline values that are only used once.
- Prefer `const` over `let`. Use ternaries or early returns instead of reassignment.
- Avoid `else`; prefer early returns.
- Avoid `try`/`catch` where possible; let errors propagate.
- Avoid the `any` type. Rely on type inference; declare types only for exports or clarity.
- Prefer functional array methods (`flatMap`, `filter`, `map`) over `for` loops; use type guards on `filter` to keep downstream inference.
- Avoid unnecessary destructuring; use dot notation to preserve context.
- Never alias imports (`import { foo as bar }`) and never use star imports.
- Use Bun APIs where possible (`Bun.file()`).
- In `src/config`, follow the self-export pattern (`export * as ConfigAgent from "./agent"`) when adding a config module.
- Drizzle: use `snake_case` field names so column names don't need redefinition.
- Comments only for non-obvious constraints or surprising behavior.

## Testing and type checking

- Tests cannot run from repo root (guard: `do-not-run-tests-from-root`). Run from package directories, e.g. `packages/opencode` or `packages/core`.
- Avoid mocks. Test actual implementation. Use `tmpdir()` + `Database.layerFromPath(tmpDbPath)` for any BanyanCode repo test that hits a real DB.
- Always run `bun typecheck` from a package directory; never `tsc` directly.
- `bun typecheck` runs `tsgo` (the TypeScript 7 native compiler, `@typescript/native-preview`), NOT the legacy JS `tsc`. This is a 4-5x speedup over the prior `tsc 5.8` pipeline and is already the default for every workspace package (`packages/*/package.json` → `scripts.typecheck: "tsgo --noEmit"`). The `typescript@5.8.2` package stays in the catalog only because `@typescript-eslint`, `@volar/typescript`, `tsconfck`, and a few codegen tools (`@hey-api/openapi-ts`, `@bufbuild/protoplugin`, `@protobuf-ts/plugin`) still require the JS `typescript` API to import — `typescript-eslint`'s peer dep is `<5.9.0` and the TS 7 `typescript` package API is officially "not ready" per [microsoft/typescript-go README](https://github.com/microsoft/typescript-go#what-works-so-far). The repository is therefore on TS 7 for typechecking but TS 5.8 for tooling imports; this is intentional and should be left alone until upstream releases a typescript-eslint version that supports TS 7.
- To force a full uncached re-run (e.g. after a `tsconfig.json` change), use `bunx turbo typecheck --force`. The pre-push hook runs the cached variant.
- To verify the per-package baseline numbers documented above, time a single package directly: `Measure-Command { bunx tsgo -p packages/core/tsconfig.json --noEmit --extendedDiagnostics }` should report ~7-8s with `Memory used: ~1.7G` for 2,557 files / 603K LOC; same command with `bunx tsc` instead of `bunx tsgo` is ~33s with ~1.3G.

## BanyanCode product identity

BanyanCode is its own product, NOT a plugin or config of OpenCode. Both install side by side and never read or write each other's files.

| Concern | OpenCode | BanyanCode |
|---|---|---|
| Per-project config | `./opencode.json` | `./banyancode.json` |
| Per-project dir | `./.opencode/` | `./.banyancode/` |
| Global config | `~/.config/opencode/` | `~/.config/banyancode/` |
| Data dir | `~/.local/share/opencode/` | `~/.local/share/banyancode/` |
| DB filename | `opencode.db` | `banyancode.db` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EkagraAgarwal/BanyanCode](https://github.com/EkagraAgarwal/BanyanCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
