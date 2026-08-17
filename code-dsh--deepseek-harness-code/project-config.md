---
trigger: always_on
description: - repository_root: `/Users/trip/TRUE 开发/deepseek/deepseek-harness-desktop`
---

# Agent Context Router

## Header

- schema_version: 3
- single_entry: true
- repository_root: `/Users/trip/TRUE 开发/deepseek/deepseek-harness-desktop`
- updated_at: `2026-08-16T20:39:00+08:00`
- default_freshness: 10d
- docs_entry: [docs/index.md](./docs/index.md)
- project_entry: [docs/project/index.md](./docs/project/index.md)
- intent_entry: [docs/project/intent.md](./docs/project/intent.md)
- knowledge_entry: [docs/knowledge/index.md](./docs/knowledge/index.md)
- plans_entry: [docs/plans/index.md](./docs/plans/index.md)

## Current Project Snapshot

- Goal: ship DeepSeek Harness Code with a cross-platform Electron shell, official-format integrated plugins, an immutable DSH Routing Suite, an optional progressive Anchored Standard Agent Preset, native Harness conversation rendering, and an independent watchdog.
- Current phase: release `0.3.3` packages the complete six-plugin Web set plus managed Skills/presets; the verified Universal DMG is available locally and PR #3 is open.
- Primary constraints: macOS Universal local release plus native Windows/Linux CI, runs on the system-installed official Node.js (>=22.13, auto-detected across common install locations on macOS, Windows, and Linux), provisions the global `dsh` command through the official npm install -g flow on first launch, loopback-only Harness, unsigned macOS distribution with ad-hoc signing.
- Active branch/worktree: `release/0.3.3-integrated-plugins` in `.worktrees/release-routing-suite`; the earlier plugin snapshot remains at `archive/desktop-plugin-before-app-merge-20260816`.
- Build/test entry: `pnpm test`; release entry: `pnpm dist:mac` then `pnpm verify:mac release/DeepSeek-Harness-Code-0.3.3-mac-universal.dmg --universal`.
- Current critical risk: the local macOS artifact is ad-hoc signed and not notarized; plugin snapshots update only with reviewed app releases, while user data and unrelated plugins remain outside the application and are never cleared by reinstall.

## User Intent Status

- status: confirmed
- confirmed_at: `2026-08-15T00:00:00+08:00`
- summary: Deliver DeepSeek Harness Code plus a checksum-pinned DSH Routing Suite and optional `anchored-standard` Agent Preset, preserve Standard and official conversation rendering, fix long-stream projection cost, and produce a verified Universal release.
- scope: desktop host, lifecycle recovery, plugin UI, DSH Routing Suite, progressive Agent Preset, managed installation, integrated packaging, performance, branding, verification.
- non_goals: auto-update, Apple notarization, cross-device warm state, reasoning-text capture/replay, private-wire mutation, or benchmark guarantees.
- acceptance: [Project intent](./docs/project/intent.md)

## Documentation Route Index

| Domain       | Read when                                                 | Entry                                        | Status |
| ------------ | --------------------------------------------------------- | -------------------------------------------- | ------ |
| Project      | Goal, scope, status, or acceptance changes                | [Project](./docs/project/index.md)           | active |
| Architecture | Process boundaries, IPC, or lifecycle changes             | [Architecture](./docs/architecture/index.md) | active |
| Engineering  | Tests, dependencies, or build conventions change          | [Engineering](./docs/engineering/index.md)   | active |
| Operations   | Packaging, installation, diagnostics, or recovery changes | [Operations](./docs/operations/index.md)     | active |
| Plans        | Multi-stage implementation work                           | [Plans](./docs/plans/index.md)               | active |
| Knowledge    | Upstream versions and official behavior                   | [Knowledge](./docs/knowledge/index.md)       | active |

## Technology Stack Index

| Technology        | Project version | Purpose                   | Status | Verified   | Details                                                    |
| ----------------- | --------------- | ------------------------- | ------ | ---------- | ---------------------------------------------------------- |
| Electron          | 43.4.0          | Desktop Chromium host     | active | 2026-08-15 | [Upstream baseline](./docs/knowledge/upstream-baseline.md) |
| DeepSeek Harness  | 0.1.0-rc.6      | Web app and agent runtime | active | 2026-08-15 | [Upstream baseline](./docs/knowledge/upstream-baseline.md) |
| electron-builder  | 26.15.3         | App and DMG packaging     | active | 2026-08-15 | [Upstream baseline](./docs/knowledge/upstream-baseline.md) |
| esbuild           | 0.25.12         | Offline plugin bundling   | active | 2026-08-16 | [Upstream baseline](./docs/knowledge/upstream-baseline.md) |
| DSH Routing Suite | pinned snapshot | Routing bundles/presets   | active | 2026-08-16 | [Upstream baseline](./docs/knowledge/upstream-baseline.md) |

## Knowledge Topic Index

| Topic             | Summary                                                                                    | Status  | Last verified | Revalidate after | Canonical document                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Code-DSH/deepseek-harness-code](https://github.com/Code-DSH/deepseek-harness-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
