---
trigger: always_on
description: Xantham is a TypeScript-to-F# bindings generator, mid-rebuild on top of `Xantham.TypeScript.Wire`
---

# AGENTS.md

Xantham is a TypeScript-to-F# bindings generator, mid-rebuild on top of `Xantham.TypeScript.Wire`
— a .NET client for TypeScript 7's own `tsc --api` compiler server that decodes its binary AST
into a typed layer. Generated bindings target **Fable 5.x only**: if a mapping needs something
Fable 5 provides, use it; a lossy mapping is lossy because of F# or a missing binding, never
because of the Fable version.

Consumer documentation lives under `docs/`; AI/agent notes live under `docs/.ai/`, indexed by
`docs/.ai/README.md`. Before changing generator behaviour, read `docs/.ai/footguns.md`: it
carries the compiler-API traps, catalog identity rules and pass invariants that earlier waves
paid for.

## Rules files

- `.claude/rules/build.md` — `build.fsx`, `tools/*.fsx`: the stage/input DSL, worktrees borrowing
  the main checkout's compiler install.
- `.claude/rules/comments.md` — repo-wide: the doc-comment style contract.
- `.claude/rules/generator-fixtures.md` — the generator, its tests, `tests/fixtures/`:
  small-lab-first workflow, the fast test loop, which generator lists are append-only.
- `.claude/rules/style.md` — `**/*.fs`: Fantomas formatting defaults and exceptions.
- `.claude/rules/upstream.md` — `tools/tsc-ast/**`: vendoring upstream TypeScript sources.

## Compile-gate dependency pin

A compile gate is only evidence if it compiles against what a consumer will: `Fable.Core` 5.2.0
(`src/Xantham.Fable.Core`, `tests/Xantham.Generator.CompileGate`), `Xantham.Fable.Core.TS`
(generated from the root compiler pin, referenced by every consumer compile gate), and the
`typescript` 7.x pin in `package.json`, which supplies both the producer's `lib.*.d.ts` and the
live `tsc --api` server.

## Project structure

- `src/Xantham.TypeScript.Wire` — the compiler client: API surface, binary AST reader, typed
  node layer, batching mailbox, virtual filesystem. Published to NuGet.
- `src/Xantham.Generator` — Harvest → Resolve → Shape → Render, one linear pass per tier
  sequenced by `Pipeline.fs`; emits a binding file plus a `manifest.json` of per-symbol findings.
- `src/Xantham.Cli` — the `xantham` .NET tool; `generate` shells over `Pipeline.run`, `schema`
  emits `xantham.schema.json`.
- `src/Xantham.Fable.Core` / `src/Xantham.Fable.Core.TS` — hand-written support library, and the
  generated ECMAScript/DOM bindings consumers open alongside it.
- `tests/Xantham.Generator.Tests` — Expecto suite including the golden corpus under `golden/`
  (`XANTHAM_UPDATE_GOLDEN=1` rewrites it — read the diff before committing it).
- `tests/Xantham.Generator.CompileGate` / `tests/Xantham.Generator.RunGate` — not test projects:
  compile the goldens, resp. Fable-run behavioural checks against a subset, on every build.
- `tests/fixtures` — real `.d.ts` packages the generator runs against, pinned in `pins.json` and
  installed by `tools/xantham-fixtures.fsx`.
- `tools/tsc-ast`, `tools/proto-gen`, `tools/session-gen` — vendor upstream sources and emit the
  AST/enum, protocol and session F# layers.
- `build.fsx` — the build pipeline (Partas.Build).

## Key commands

- `dotnet build Xantham.slnx` — build. `dotnet test` — run the Expecto suite.
- `dotnet fsi build.fsx -- <build|test|generate|docs|pack|publish|bump>` — full pipeline.
- `dotnet fsi build.fsx -- generate [--only ast|proto|session|compiler-lib|schema] [--sync]` —
  routes to `tools/generate-wire.fsx`; `--sync` re-vendors upstream sources first (network).
- `dotnet run --project src/Xantham.Cli -- generate <package-dir> [-o <dir>] [--config <path>]` —
  generate a binding outside the test harness.
- `dotnet fsi tools/xantham-fixtures.fsx -- init` — install `tests/fixtures/` from `pins.json`;
  also runs as a stage of `build.fsx -- test|pack|publish`.

An agent worktree under `.claude/worktrees/` carries tracked files only; `build.fsx` borrows the
main checkout's compiler install rather than reinstalling (`.claude/rules/build.md`), while
fixtures are expected to be downloaded fresh there (`.claude/rules/generator-fixtures.md`).

## F# semantics — use `fslangmcp`, not grep

The repo ships an `fslangmcp` MCP server (`.mcp.json`) loaded against `Xantham.slnx`. **Answer
semantic questions about F# code with it; reach for grep only for prose, JSON, `.mts` tooling and
other non-F# files.** Over 26k lines of `*.generated.fs` sit in `src/`, so textual search
over-matches badly — short names like `decode` or `is` recur in thousands of generated members.

- `find` — definitions and cross-project use sites. **Run `check` first**: when the workspace does
  not type-check, `find` can return `outcome="not_found"` with `coverage.complete: true` — a
  confidently wrong negative. Never trust a `find` taken while `check` reports errors.
- `check` — fresh whole-workspace type-check verdict with structured diagnostics.
- `fcs_refactor_impact` — run before changing any public signature: blast radius, whether the
  symbol is public API, covering tests.
- `fcs_tests_for_symbol` — tests by direct call site; an empty result means "not called by name",
  not "untested" (much of the suite exercises the library through the wire).
- `fcs_public_api`, `fcs_nuget_types`/`fcs_nuget_members`/`fcs_referenced_symbols` — public surface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shayanhabibi/Xantham](https://github.com/shayanhabibi/Xantham) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
