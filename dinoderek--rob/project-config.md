---
trigger: always_on
description: Roblox game written in native Luau. This file is the canonical instruction set
---

# rob — agent instructions

Roblox game written in native Luau. This file is the canonical instruction set
for ALL coding agents (Claude Code reads it via CLAUDE.md; Codex reads it
directly). Keep it up to date: when you change tooling, structure, or policy,
update this file in the same commit.

## Project map

```
src/shared/    Pure Luau only. No Roblox globals (game, Instance, task, …).
               Runs under Lune. This is where game logic lives.
src/server/    Services/  — pure orchestration, all effects behind injected deps (Lune-testable)
               Adapters/  — thin wrappers around Roblox services (DataStore, …); NO logic
               init.server.luau — composition root; the only place adapters get wired in
src/client/    Thin rendering/input glue. No game logic.
tests/         Lune unit tests (*.spec.luau) + fakes in tests/helpers/
lune/          Task scripts (`lune run <name>`) + vendored libs in lune/lib/
docs/          Design notes, including the cloud integration test stub
```

Layer rules (enforced by review, not tooling — do not break them):

1. `src/shared/**` and `src/server/Services/**` must never touch a Roblox
   global. If code needs a Roblox service, define a minimal structural
   interface (see `StoreLike` in `CurrencyService.luau`), implement it in
   `src/server/Adapters/`, and inject it from the composition root.
2. Adapters contain zero logic — one Roblox API call per method, nothing else.
   Logic in an adapter is logic you can't test.
3. Composition roots (`init.server.luau`, `init.client.luau`) only wire things
   together. If a root grows logic, extract it into a service.

## Quality gates — always run them

**Before considering ANY work done, run `lune run check` from the repo root
and get every gate green.** No exceptions, including "trivial" changes.

| Gate | Command | What it catches |
|---|---|---|
| format | `stylua --check src tests lune` | style drift |
| lint | `selene src tests lune` | undefined globals, suspicious code |
| types | `lune run analyze` | strict Luau type errors vs real Roblox/Lune APIs |
| unit tests | `lune run test` | behavior regressions |
| build | `rojo build default.project.json --output build.rbxl` | invalid project/instance tree |

`stylua src tests lune` (no `--check`) auto-fixes formatting.
`lune run test -- --update-snapshots` refreshes tiniest snapshots.
CI (`.github/workflows/ci.yml`) runs the same gates — if it's red there, it
was runnable locally first.

## Guardrails

- **Never trust your memorized knowledge of library versions or APIs.** The
  Roblox toolchain moves fast and your training data is stale. Before adding,
  upgrading, or writing code against any tool or library: check the actual
  GitHub releases page / registry entry / `--help` output / bundled type
  definitions, and pin exact versions. This applies to Roblox engine APIs too
  — verify against the type definitions that `lune run analyze` downloads
  (`.cache/globalTypes.d.luau`) rather than assuming a method exists.
- **All tools are pinned in `rokit.toml`.** Install with `rokit install`;
  never install ad-hoc tool versions or invoke tools not listed there. To
  upgrade a tool: verify the new version's release notes, bump `rokit.toml`,
  run all gates, and note anything that changed behavior.
- **Do not commit generated artifacts**: `build.rbxl`, `sourcemap.json`,
  `.cache/` are gitignored — keep it that way. `roblox.yml` IS committed on
  purpose (pinned selene std; refresh with `selene update-roblox-std`).
- **Never edit `lune/lib/**` (vendored third-party code).** See
  `lune/lib/tiniest/VENDOR.md` for how to update it.
- **Secrets never enter this repo.** Roblox Open Cloud API keys live in
  GitHub Actions secrets / local env vars only.

## Require rules (important — two runtimes, one style)

- Inside `src/**`: **relative string requires only** — `require("./Sibling")`,
  `require("../../shared/Currency")`. In an `init.luau`, a child of the module
  is `require("@self/Child")`. These resolve identically in the Roblox engine
  (require-by-string) and Lune because `default.project.json` mounts
  `src/shared` as a sibling of `src/server` inside ServerScriptService.
  Do NOT use `.luaurc` aliases in `src/**` — engine support for aliases is not
  established.
- Inside `tests/**` and `lune/**` (Lune-only code): aliases from `.luaurc` are
  fine and preferred: `@shared/...`, `@server/...`, `@tiniest/...`, plus the
  Lune builtins `@lune/fs`, `@lune/process`, etc.
- Composition roots and adapters may use classic instance access
  (`game:GetService(...)`) — they are Roblox-only glue.

## Luau strict-mode notes

- All code is `--!strict` (set project-wide in `.luaurc`).
- Don't compare metatable-based types (e.g. `Wallet`) against `nil` with
  `==`/`~=` — strict Luau rejects it ("do not have the same metatable").
  Use truthiness: `if not wallet then`.
- Local runs of `lune run check` may skip the types gate if luau-lsp isn't
  installed; CI always runs it. Don't treat "passed locally" as green if the
  types gate was the one that failed to run.

## Testing policy

- Every module in `src/shared/**` gets a spec in `tests/**` mirroring its
  path (`src/shared/Currency/Wallet.luau` → `tests/Currency/Wallet.spec.luau`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dinoderek/rob](https://github.com/dinoderek/rob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
