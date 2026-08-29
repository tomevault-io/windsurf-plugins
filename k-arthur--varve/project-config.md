---
trigger: always_on
description: Local-first, cross-platform design suite. Native Rust engine on desktop
---

# AGENTS.md — Varve

Local-first, cross-platform design suite. Native Rust engine on desktop
(Tauri 2), WASM behind the same facade on web. Linux (CachyOS/Arch) is the
primary dev OS.

## Toolchain (confirmed working, 2026-06-29)
- Rust: `~/.cargo/bin` (rustc 1.97 / cargo 1.97). Source with `. "$HOME/.cargo/env"`.
- pnpm 11.9: `~/.local/share/pnpm/bin`. Export `PNPM_HOME="$HOME/.local/share/pnpm"` and add `$PNPM_HOME/bin` to PATH.
- just 1.58: `~/.local/bin`.
- Node 26, npm 11.16.
- wasm32 target installed.
- WebKitGTK 2.52.4 / GTK 3.24.52 / librsvg / openssl / fontconfig / fuse2 confirmed via pkg-config.
- Optional: `cmake`, `xdotool` (not needed for core build).

## Validation economy — mandatory

**Validate according to impact, not repository size.** The full suite is an
explicit escalation/final-gate operation, not the default inner loop. Canonical
policy: `docs/quality/validation-strategy.md`. Executable planner:
`scripts/quality/affected-plan.mjs`.

1. Inspect `git status` / diff first.
2. Run `pnpm verify:plan` — the planner explains exactly what it selects
   and why, and what it deliberately skips.
3. Run `pnpm verify:affected` (Tiers 0–4, dependency-aware; `pnpm verify:quick`
   for trivial edits).
4. Add feature-specific E2E/visual/perf validation when the plan indicates.
5. Do NOT run `pnpm test`, `just gate`, full Playwright, or
   `cargo test --workspace` by default.
6. Full validation (`pnpm verify:full`) is reserved for explicit escalation
   conditions — workspace/toolchain changes, test-runner configuration,
   serialization/schema migrations, cross-package foundational API changes,
   release checkpoints, explicit request — and requires a stated reason.
7. Record exactly what was run, what passed, and what was skipped as
   unrelated (the Agent Validation Report template, below).

### Failure-resolution and release candidates

For a large integration or release candidate, do not restart a broad E2E
gate after every individual repair. Run `pnpm verify:triage` once to collect
a bounded failure set. Triage still runs the affected closure when the
planner requires a final full gate, exposing downstream compile failures
before that expensive checkpoint. Classify and repair each failure, and run its affected
compiler check, exact spec, and direct unit checks after each repair. Changes
under `tests/e2e/` automatically select `pnpm typecheck:e2e` before
Playwright. When the candidate is frozen, run `pnpm verify:affected` once
and then the required release `pnpm verify:full` gate. Shared E2E
infrastructure (`tests/e2e/shared.ts`,
`tests/e2e/helpers/**`, and fixtures), runner configuration, renderer, and
other integration surfaces deliberately retain broad validation scope.

Skipping affected tests is prohibited. Running unrelated tests is
discouraged — it consumes shared developer resources and delays feedback.

### Agent Validation Report (before claiming completion)

```text
Changed scope: <files / packages / crates>
Validation plan: <summary of pnpm verify:plan output>
Commands actually run: <exact commands>
Passed: <list>
Skipped as unrelated: <list + why>
Escalations: <list>
Full suite run: yes/no
If yes, reason: <stated reason>
```

## Regression protocol (mandatory after every architecture/system change)

After ANY change that touches:
- type definitions, interfaces, or generics
- imports, re-exports, or module boundaries
- function signatures or overloads
- state shape or context values
- test files or test infrastructure

Run in order:
```bash
pnpm verify:plan        # understand the impact first — never skip this
pnpm verify:affected    # Tiers 0-4. Tier 0 already covers format+lint on the
                        # touched files — no whole-repo format/lint runs per
                        # change; those belong to the full gate only.
pnpm bench              # benchmark mode for .bench.ts files (optional, perf-sensitive)
pnpm audit:docs         # docs naming/index/link drift — zero violations
pnpm audit:emoji        # zero violations
pnpm audit:tokens       # 120/120 WCAG-AA (3 themes)
```

The audits above are also auto-selected by the planner when the change
touches their domain (docs/emoji/tokens) — running them directly is a
belt-and-braces recheck, not a substitute for `verify:affected`.

Failure at any step means the change introduced a regression. Fix before committing.
Do NOT skip the affected checks — but do NOT run the full suite unless the
planner escalated (`FULL-SUITE ESCALATION: YES`) or a full-gate checkpoint
applies. When the planner escalates, run:

```bash
pnpm verify:full    # requires VARVE_FULL_GATE_REASON (see docs/quality/validation-strategy.md)
```

### Code-health check (triage gate)

After any system-level change (new context provider, new hub file, new tool system,
or any change to `context.tsx` / `CanvasArea.tsx` / `Shell.tsx`), run the
architecture audit to verify architectural metrics haven't regressed:

```bash
node scripts/audit-architecture.mjs --ci
```

These numbers drift with everyday merges — treat this table as orientation
and the audit output plus the committed baselines as the source of truth.
Enforced by `audit-architecture.mjs --ci` against `.architecture-baseline.json`
(live values 2026-08-22):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [K-Arthur/varve](https://github.com/K-Arthur/varve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
