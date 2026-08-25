---
trigger: always_on
description: Mobile remote editor for iOS and Android. Primary platform is iOS (`gpui_ios` + Metal). Secondary platform is Android (`gpui_android` + `gpui_wgpu` + Vulkan).
---

# Zedra

Mobile remote editor for iOS and Android. Primary platform is iOS (`gpui_ios` + Metal). Secondary platform is Android (`gpui_android` + `gpui_wgpu` + Vulkan).

`docs/CONVENTIONS.md` is canonical for code style, error handling, logging, git commits, PRs, docs style, theming, icon usage, async runtime selection, GPUI entity/task/layout rules, and `WorkspaceState` data flow. This file does not repeat them.

## Repo Map

- `crates/zedra` — mobile app: GPUI UI, workspace orchestration, platform bridge, iOS/Android native glue
- `crates/zedra-host` — desktop daemon + `zedra` CLI: iroh endpoint, auth, sessions, PTY, fs/git, managed agents
- `crates/zedra-session` — client-side connect/reconnect, auth, session events, remote terminal attach
- `crates/zedra-terminal` — reusable terminal emulator + GPUI renderer (alacritty model, input/IME, OSC)
- `crates/zedra-rpc` — irpc protocol types and QR pairing between client and host
- `crates/zedra-osc` — packet-safe OSC scanner for PTY byte streams
- `crates/zedra-telemetry` — typed telemetry events with per-runtime backend injection
- `ios/`, `android/` — native app shells (xcodegen project, Gradle)
- `packages/` — TypeScript workspaces, formatted by biome: `landing` (Astro/Starlight site for zedra.dev), `relay-check` (CLI), `relay-monitor` (Dockerized relay health monitor)
- `scripts/` — build/run/log/asset/setup tooling; `vendor/zed` — patched GPUI submodule
- `crates/zedra`, `zedra-host`, `zedra-session`, and `zedra-terminal` each have their own `AGENTS.md` — read it before working in that crate.

## Everyday Commands

- First-time setup: `git submodule update --init --recursive`; full prerequisites in `docs/GET_STARTED.md`.
- Host CLI (binary is named `zedra`): `cargo run -p zedra-host -- start|status|qr|setup|agent|logs ...`. The daemon is per-workspace — a workspace lock refuses a second `start` in the same workdir; use `start --detach` for background runs.
- iOS app: `./scripts/run-ios.sh [sim|device]`; logs via `./scripts/ios-log.sh tail`. Details: `docs/IOS_WORKFLOW.md`.
- Android app: `./scripts/run-android.sh` (never `adb shell am start` directly); logs via `./scripts/log-android.sh`.
- `zedra setup` flows: verify in the throwaway sandbox — `scripts/setup-sandbox.sh zedra setup <agent>` (shimmed provider CLIs, no network, real `$HOME` untouched).
- Icons: `bun run icons:gen` after adding an SVG.

## Agent Workflow

- Inspect the relevant code paths first and infer local patterns before proposing or making changes. Read the owning crate's `AGENTS.md`.
- When a request mentions the Zedra CLI or `zedra` command-line behavior, inspect `crates/zedra-host` first; that crate owns the daemon and CLI entrypoints.
- Ask before making any meaningful product or architectural decision. Tiny details may follow existing patterns without approval.
- Prefer the smallest diff that fits the current design. If the structure is blocking quality, propose the refactor and wait for approval.
- Keep code concise and modular; prefer clarifying code over comments. Comments stay 1 line (up to 3 only for genuinely confusing parts), same for doc comments. Add a minimal comment at regression-prone guards explaining the invariant.
- Prioritize correctness and clarity over cleverness or speed unless performance is the explicit problem.
- Avoid panic-prone shortcuts such as unchecked indexing or `unwrap()` in normal code paths. Propagate or handle errors.
- Surface blockers quickly with a recommendation. Keep progress updates short and include reasoning or tradeoffs.

## Debugging Workflow

- Read the relevant code path deeply before changing behavior.
- On mobile issues, prefer targeted `tracing` logs with a clear searchable prefix so the developer can run the app, reproduce, and return logs.
- After the first failed debugging attempt, stop and ask for more information instead of arguing from hypotheses.
- Prefer root-cause fixes once the issue is confirmed.

## Repo Invariants

- `WorkspaceState` is the single source of truth for display state; `render()` stays pure; platform access goes through `platform_bridge::bridge()`; logging uses `tracing`, never `log::`. Details and data flow: `docs/CONVENTIONS.md`.
- Read `docs/DESIGN.md` before creating or redesigning UI, and `docs/THEMING.md` before adding colors — `theme::` tokens only, no hardcoded hex in views.
- GPUI tasks cancel when the `Task` handle drops. Inside entity `update`/`read_with` closures, use the inner `cx`; reentrant updates panic.

## Managed Agents

- `docs/MANAGED_AGENTS.md` is canonical for adding or changing AI-agent support. The host actor is the source of truth; the app adapter is optional branding/behavior.
- One `AgentActor` per agent in `crates/zedra-host/src/agent/<slug>.rs`, registered in the `ACTORS` array in `agent/mod.rs`. Never add per-agent `match` arms to the REST API, host cache, CLI scans, hook dispatch, or installed-agent list — everything resolves through the registry.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deltaqdev/zedra](https://github.com/deltaqdev/zedra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
