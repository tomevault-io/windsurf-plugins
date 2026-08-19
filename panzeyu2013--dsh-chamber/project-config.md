---
trigger: always_on
description: dsh-chamber is the local desktop **connection manager** for dsh: the local dsh instance (web profile) is hosted by the control plane; dsh instances on remote servers are attached over SSH tunnels. The UI is the **dsh official frontend, source-reused and self-built** (single window, single frame; multiple instances coexist as N-ctx shells). The control plane owns connection management, per-instance same-origin reverse proxying, and static frontend serving (v1 has no authentication/audit surface —
---

# dsh-chamber Agent Guide

## Purpose

dsh-chamber is the local desktop **connection manager** for dsh: the local dsh instance (web profile) is hosted by the control plane; dsh instances on remote servers are attached over SSH tunnels. The UI is the **dsh official frontend, source-reused and self-built** (single window, single frame; multiple instances coexist as N-ctx shells). The control plane owns connection management, per-instance same-origin reverse proxying, and static frontend serving (v1 has no authentication/audit surface — removed with the 2026-08-14 consolidation). Host-native capabilities (goals, jobs, terminals, schedule, settings, pluginInventory, …) remain the host and host-frontend's job — the control plane only attaches, never re-implements. **Session business is entirely the dsh frontend runtime's job; the control plane consumes no host frames.**

This file contains only always-on repository rules and routing. Detailed design lives in `docs/design/`, progress in `docs/progress/`, and unimplemented feature ideas in `docs/todo/`.

> 中文版见 [docs/AGENTS.zh-CN.md](docs/AGENTS.zh-CN.md)

## Instruction Order

These steps are mandatory. Before editing, you **MUST**:

1. Follow this root guide.
2. Read the relevant design document (`docs/design/0X-*.md`) and the progress overview (`docs/progress/STATUS.md` — the only progress record).
3. Follow the consolidation principles of `docs/design/01-overview.md`: anything the dsh host or its frontend already covers is attached/served by the control plane — **never re-implement an execution surface**; the control plane does not consume host sessions; domains removed from scope (walkthrough, notifications, MCP, thin-shell chat UI, control-plane session runtime, …) **must not return in any form**. One carve-out (2026-08-16, design 08): git/GitHub stays out of the control plane and the 本体, but may ship as a chamber-bundled client plugin (worktree lifecycle etc., see 08) — the plugin never re-implements a dsh host surface.
4. Honor documented deviations in module docs.

If these sources materially conflict, stop and resolve the conflict instead of silently choosing one.

## Runtime Boundaries

- `packages/control-plane` — connection-manager core: local host hosting (web-profile spawn/readiness/reaper/health/logs), management REST (`/health`, `/api/connections` local-only, `/api/host/logs`), per-instance generic reverse proxy (`/api/i/<id>/*` HTTP/WS/SSE passthrough; v1 anonymous, loopback-only — no auth/audit surface), static frontend serving (dist + `__DSH_BOOT__` manifest).
- `packages/renderer` — the self-built dsh frontend (source reuse): entry build (chamber composite entry), the pure-dsh first screen bridge host (entry-level React: local auto-start / registry auto-connect / chamberBridge publish & onOpenSession), N-ctx multi-instance orchestration, boot manifest generation, design 09 module C (per-instance host-graph merge + extra-entry preloading: `host-graph.ts` + `chamber-covered.ts`).
- `packages/dsh-client-connection` — in-repo copy of the official connection client with the base-path patch (shadows the vendored workspace entry).
- `packages/dsh-client-web` — in-repo copy of the official web shell with the boot.tsx N-ctx module-table sharing seam (incl. the per-instance `extraRows` boot-row merge, design 09 module D) + runtimeCtx getter (shadows the vendored workspace entry).
- `packages/dsh-chamber-client-ui-sidebar` — the self-built sidebar plugin (copied ui-sidebar structure): multi-source session navigation + chamberBridge (`shared/aggregate-store.ts` + per-instance unary client `shared/instance-api.ts`), replacing the official ui-sidebar registration (see 05 §6).
- `packages/dsh-chamber-client-ui-settings-connections` — the self-built connections settings plugin: local instance card + remote host CRUD/connect/systemd/logs (settings.section, dsh design tokens, see 05 §5).
- `packages/dsh-chamber-client-ui-settings-bridge` — the self-built settings SHELL plugin: replaces the official SettingsRoot registration (sidebar.settings at priority -1, shadowing) — a server dropdown over the selected instance's official settings sections (child cordis context bridge) plus the fixed chamber-global connections nav entry (see 05 §5 sibling design discussion 2026-08).
- `packages/desktop` — Electron shell: single frame (`loadURL` the control-plane origin), transport-manager (generic transport runtime; `transport-provider.ts` interface + the `ssh` provider in `ssh-provider.ts` — tunnels + systemd exec, v1 kind `ssh`), instance registry (`<userData>/ssh-instances.json`), IPC.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panzeyu2013/dsh-chamber](https://github.com/panzeyu2013/dsh-chamber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
