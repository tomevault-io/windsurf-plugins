---
trigger: always_on
description: This is the root devlog contract for the `ocean-os` repository. Every agent entering this repo — Claude, Codex, Pi, ocean-native, or any other harness — reads this file first.
---

# Ocean OS — Devlog Root

## Purpose

This is the root devlog contract for the `ocean-os` repository. Every agent entering this repo — Claude, Codex, Pi, ocean-native, or any other harness — reads this file first.

## Ownership

- **Repo:** `risingtides-dev/ocean-os`
- **Runtime:** Rust workspace, daemon on `:4780`, TUI binary `ocean`
- **Connected Ocean repos:** route cross-repo work through `docs/OCEAN_PROJECT_MAP.md`; do not infer ownership from proximity.

## Local Contracts

- Read this file before editing anything in this repo.
- Walk from repo root to each target path and read every `AGENTS.md` along the route.
- Use the nearest `AGENTS.md` as the local contract; parent docs set repo-wide rules.
- No child doc may weaken this root contract.
- Cross-repo routing map: `docs/OCEAN_PROJECT_MAP.md`.
- Canonical workspace package/entry/test index: `crates/AGENTS.md`. Do not maintain a second crate inventory here.
- After any meaningful change, do a devlog pass: update the nearest owning `AGENTS.md`, refresh affected child indexes, remove stale text, and append a root `events.md` entry with `worktree:`.
- Project code is `MIT OR Apache-2.0`, copyright © 2026 Rising Tides. Preserve `NOTICE.md` and local donor notices; Ocean names, logos, and distinctive brand assets remain outside the open-source grants under `TRADEMARKS.md`.
- Rust workspace packages are private distribution inputs, not crates.io releases: `[workspace.package] publish = false`, and every member must inherit it or set `publish = false` explicitly. `cargo xtask docs-check` enforces this for new members.

## Workspace Routing

Core execution flow:

```text
clients (TUI / CLI / ACP / surface)
  -> ocean-daemon (HTTP/SSE authority)
  -> ocean-agent (sessions, prompts, capability assembly)
  -> ocean-runtime (agent loop, permissions, tools)
  -> ocean-protocol + ocean-providers (wire encoding + model/auth routing)
```

Use `crates/AGENTS.md` for all current workspace packages, ownership exclusions, entry points, local contracts, and narrow validation commands.

## Work Guidance

- Current architecture and operations live in `docs/ARCHITECTURE.md` and `docs/OPERATIONS.md`; open work lives in `ROADMAP.md`.
- The active extension architecture and staged migration program is governed by `docs/specs/2026-07-14-ocean-extensions-architecture-and-migration-manifest.md`; Phases 0–1 are accepted. The operator-accepted Phase 6 orchestration-transport ratification is `docs/specs/2026-07-18-ocean-crew-orchestration-and-durable-workflow-manifest.md`; the exact Stage A implementation contract at `docs/specs/2026-07-27-ocean-extension-stage-a-implementation-manifest.md` authorizes only its ordered A1 → A2a → A2b → A3a → A3b → A4 → A5 sequence. A1, A2a, and A2b are accepted. A2b supplies authoritative metadata-only lifecycle publication, synchronous project snapshots and activation epochs, bounded replay/live delivery with fixed ACK state, checked project reconciliation, health/restart/circuit policy, and retained generation-safe cleanup without registry mutation, routes, CLI, or A3+ behavior. A3a is the next authorized slice and alone may atomically add the durable first-Stage-A-publication marker that makes later `service-grants.json` absence fail closed. Crew Stages B–E each require their own implementation manifest and keep extension-provided Longhouse facades over an extension-owned Crew engine, not core.
- The Ocean Observatory program is governed by `docs/specs/2026-07-17-ocean-observatory-architecture.md` with Gate 0 accepted in `docs/specs/2026-07-17-observatory-gate0-decisions.md` (operator ruling: 90s pixel-game visual parity on truthful real events with a durable event store; security invariants unchanged). The operator accepted the Gate 1 implementation manifest at `docs/specs/2026-07-17-observatory-gate1-implementation-manifest.md` on 2026-07-17; tasks 2–9 are authorized under its strict dependency order and stop conditions. Tasks 2–8 are landed; the Task 9 independent review is retained at `docs/specs/2026-07-20-observatory-gate1-task9-independent-review.md` with gating repairs G1–G5 that must land and pass delta review before production Ocean Floor renderer work. Do not build from the Claude Design export code or the raw global agent firehose.
- The Ocean Browser program is governed by `docs/specs/2026-07-19-ocean-webkit-browser-program.md` (operator ruling 2026-07-19: a custom WebKit engine with earned Chrome DevTools protocol parity, built outside the Cargo graph in a dedicated repository). The legacy Chromium backend is quarantined behind the default-off `legacy-chromium` feature; the supervised daemon keeps interim browsing via `ops/install-ocean-daemon.sh`. Acceptance gates are fixed in the manifest; do not ship partial-fidelity network capture as parity.
- The active behavior-neutral daemon refactor is governed by `docs/DAEMON_REFACTOR_MISSION.md` and the supporting code-health plan under `docs/specs/`.
- Optimize for cold-agent discoverability: ownership, entry point, critical invariant, and narrow validation must remain findable from the root, `docs/README.md`, and `crates/AGENTS.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Risingtides-dev/ocean-os](https://github.com/Risingtides-dev/ocean-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
