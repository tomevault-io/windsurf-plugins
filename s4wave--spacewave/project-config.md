---
trigger: always_on
description: This is the repository-local guide for Spacewave. If a broader workspace
---

# Spacewave Agent Guide

This is the repository-local guide for Spacewave. If a broader workspace
rulebook is already in force, keep following it; this file adds Spacewave's
product model, package boundaries, recurring hazards, and verification commands.
If this is the only guide available, read it with `README.md`, `DESIGN.md`,
`package.json`, `go.mod`, and the files near the target code before editing.

## Entry Contract

- State the work surface and proof event before editing. For behavior-changing
  work, name the owner, invariant, falsifier, proof command, and adjacent risk.
- Ask before branch changes, commits, pushes, releases, live-service mutation,
  credential handling, destructive cleanup, public API changes, durable data
  breaks, or behavior changes that materially affect lifecycle, concurrency,
  persistence, sync, latency, or user-visible behavior.
- Preserve concurrent work. Treat dirty files you did not edit as user or agent
  work; inspect before depending on them and never revert them unless asked.
- Keep private planning artifacts, phase labels, process notes, and internal
  process references out of Spacewave code, comments, commits, PRs, and public
  docs.
- Prefer the smallest complete owner-level fix, not the smallest diff. Delete
  speculative code and shallow helpers when they sit inside the touched owner.

## Product Model

Spacewave is a local-first Go and TypeScript application framework for
peer-to-peer collaborative apps on Hydra and Bifrost. The shared substrate is
Spaces, SharedObjects, storage/sync, Spacewave Cloud, PluginHost, and Bldr
plugin loading. Focused apps and plugins are front doors over that substrate;
they must not fork account, Space, SharedObject, storage, sync, cloud, or SDK
semantics.

Runtime reach must not fork the product model:

- Go owns SDK/core semantics and native runtime paths.
- Browser Go plugins target `web/js/wasm` through Bldr/TinyGo and run in browser
  workers.
- TypeScript plugins target `js` through WebWorker, QuickJS, and browser QuickJS
  paths.
- GoScript is a selected-package Go-to-TypeScript compiler for browser builds
  and staging gates, not a guarantee that every Go package has GoScript parity.
- Every Bldr plugin has its own bus. Cross-plugin behavior uses explicit
  RPC/resource/plugin-host boundaries; do not assume directives cross buses.
- A frontend `entrypoint=True` JS plugin can still render its root WebView
  across separate `-core`/`-web`/`-app` plugins. Use the existing
  `-core`/`-web`/`-app` layout for app/plugin structure questions.

Spacewave is alpha with real users. Clean breaks are preferred inside the repo,
but never silently break durable local state, cloud state, database migrations,
wire/storage formats, or user-owned data. Surface the reset/migrate/compat
choice first.

## Owner-Level Defaults

- Fix the owner of the invariant, state machine, lifecycle, resource, cache,
  storage, wire format, or UI data flow. Do not add caller-side guards that
  reconstruct owner state.
- A GoScript codegen or typecheck failure is a GoScript compiler bug. Fix it in
  the `github.com/s4wave/goscript` compiler (lowering, runtime override, or
  barrel emission) with a compliance fixture, then bump the `go.mod` pin. Never
  work around it by reshaping the spacewave Go source to dodge the compiler,
  hand-editing generated `.gs.ts`, or swapping to a different API only because
  GoScript mistranslates the first. The spacewave source stays idiomatic Go.
- No fixed-interval polling for state. Owners expose waits, watches, broadcasts,
  conditions, event streams, or context-aware blocking calls.
- No fire-and-forget goroutines from callbacks, handlers, WebSocket frames, or
  hot paths. Use `util/routine`, `util/keyed`, `util/refcount`, and
  `broadcast.Broadcast` under an owning lifecycle context.
- Do not add dead-code fallback paths for impossible conditions. If construction
  guarantees a field, enforce construction and let violations fail at the owner.
- Comments record durable invariants, contracts, boundaries, algorithms, or
  historical failures only. Do not narrate the code, task, or session.
- Docs describe the current system in present-state wording, not migration
  history, phase history, or temporary implementation notes.

## Repository Commands

Run commands from the repository root unless a package README or script requires
a subdirectory.

- Use `bun run ...` for JS/TS/package scripts; do not use `npm`, `yarn`, `pnpm`,
  `npx`, or bare package binaries from the shell.
- Run `bun install` before treating generated/module-resolution failures as real.
  Use `bun run setup` only to repair stale `.bldr` exports or module resolution
  after dependencies are installed.
- Do not edit, copy into, or sync files under `.bldr/src/`; it is generated.
  Edit source files in their original locations.
- Keep large command output under `.tmp/` when needed, then inspect a short
  summary or tail. Do not commit `.tmp/`.
- Do not use sleep loops for readiness, locks, files, ports, pids, or results.
  Prefer one blocking command with a real timeout or an event-backed background
  run whose completion is reported once.
- Do not add new prototypes in this repo unless explicitly asked and the target

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s4wave/spacewave](https://github.com/s4wave/spacewave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
