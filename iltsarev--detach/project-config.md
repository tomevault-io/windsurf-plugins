---
trigger: always_on
description: Detach is a macOS 26+ reliability harness for persistent Codex CLI and Claude
---

# Detach agent guide

Detach is a macOS 26+ reliability harness for persistent Codex CLI and Claude
Code sessions. It owns a private tmux runtime, recovery checkpoints, typed
state, an app, and two-layer power protection. Users install and authenticate
the providers separately.

## Start here

1. Inspect the working tree; preserve unrelated user changes.
2. Use the context map below to read the one relevant specification. Read a
   second spec only when the change genuinely crosses that boundary.
3. Edit a small, obvious change directly. For cross-subsystem work, a risky
   migration, or unresolved requirements, create an ignored ExecPlan under
   `docs/work/`. Before implementation, write and review its current-to-target
   contract delta. Ask the owner only about material choices the task does not fix.
4. During implementation, run the narrow checks named by the selected specs.
   Before handoff, inspect `scripts/quality-gate --plan --explain` and run the
   focused local diagnostics once. Use `--resume latest` after a compatible
   interrupted or failed run. Hosted pull-request CI is readiness authority.
5. Review the final diff and evidence. Update the user contract or durable spec
   in the same change whenever behavior or an invariant changes.
6. Unless the owner asks to keep work local, use a topic branch. Stage only
   task-scoped files, inspect the staged public diff, and summarize the safe
   contract delta, durable decisions, and evidence in the PR. Merge only after
   its authoritative `quality-gates` job passes. Verify final `main` upstream parity.
   Release metadata uses a PR. `scripts/release-version` is the sole entry.

`README.md` is the user-facing contract. `docs/specs/` contains durable
current-state engineering contracts. Tests and gates are executable evidence;
they do not make stale prose correct.

Use [ASD-STE100 Issue 9](https://www.asd-ste100.org/) for changed English in
`README.md` and `docs/`. Use short, direct sentences and one term per meaning.
Product names, paths, commands, and identifiers are technical terms. Do not
claim verified compliance without review against the official standard.

## Context and specification policy

- Keep this file under 200 lines and limited to common rules.
- Put durable invariants in the narrowest `docs/specs/` file and temporary plans
  in `docs/work/`. Keep tutorials and inventories out.
- `CLAUDE.md` must contain only `@AGENTS.md`. Never copy this content
  into it or create a second lowercase agent-instruction file.
- Do not import detailed specs: Claude loads imports eagerly. Use the context
  map below.
- Specs state observable outcomes, non-goals, invariants, owning paths, and
  verification. Avoid code narration.
- Complex plans are living, self-contained handoff artifacts. Record decisions,
  discoveries, progress, and end-to-end evidence; delete or archive obsolete
  local plans when the task ends.
- If a correction repeats, encode it at the narrowest durable layer: executable
  check first when possible, then a scoped spec, and only then this file.

## Context map

| Change | Read | Fast feedback |
| --- | --- | --- |
| Runtime CLI, lifecycle, install, tmux | `docs/specs/runtime.md` | `tests/{run,run-claude,distribution}.sh` |
| State, storage, events | `docs/specs/state.md` | one Swift state filter |
| Power, helper, watchdog | `docs/specs/power.md` | one Swift Power filter |
| App UI, terminal, UI smoke | `docs/specs/app.md` | one Swift app filter |
| Setup, settings, diagnostics, updates | `docs/specs/app-setup.md` | one Swift app filter |
| Package, release, publication | `docs/specs/release.md` | `tests/{release,publish}-*.sh` |
| Docs, specs, test workflow | `docs/specs/documentation.md` | `tests/docs-contract.sh` |

For unfamiliar or cross-cutting work, start at `docs/specs/README.md`. Do not
read every spec.

## Verification loop

- `scripts/quality-gate --plan --explain`: inspect the local diagnostic stages
  selected from the actual diff.
- `scripts/quality-gate`: impact-aware local diagnostic.
- `scripts/quality-gate --mode repository`: every automated repository check
  as a local diagnostic. Hosted pull-request CI runs this mode as authority.
- `--stage <name>` and direct test commands are diagnostic only, not readiness
  evidence.
- Prefer one focused test while iterating. Do not repeatedly pay for the full
  suite when a narrower deterministic check can close the feedback loop.
- Stage timing is telemetry, not a verdict. Treat a slow stage as performance
  work. Never rerun merely for warmer caches, timing variance, or a lucky
  result; rerun unchanged only after evidence identifies an unrelated external
  transient, and record its cause.
- Never run real power tests, signing, notarization, tagging, upload, or
  publication during ordinary implementation.

See `docs/testing.md` for commands, evidence/resume semantics, and manual
release-only checks. See `docs/quality-gates.md` for the gate policy.

## Non-negotiable product invariants

- Runtime payloads are immutable and self-contained under
  `~/.local/libexec/detach/versions/<semver>-<hash>/`; production never
  falls back to ambient tmux, jq, or Homebrew helpers.
- `bin/detach` is the public CLI. `bin/detach-core` owns lifecycle and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iltsarev/detach](https://github.com/iltsarev/detach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
