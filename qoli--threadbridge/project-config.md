---
trigger: always_on
description: This root `AGENTS.md` is the maintainer guide for `threadBridge`. It documents the repo layout, runtime boundaries, workspace lifecycle, and contributor conventions for the Telegram bot and its Codex app-server integration.
---

# Repository Guidelines

## Purpose
This root `AGENTS.md` is the maintainer guide for `threadBridge`. It documents the repo layout, runtime boundaries, workspace lifecycle, and contributor conventions for the Telegram bot and its Codex app-server integration.

It is not the runtime skill used inside a bound project workspace. That skill lives in [runtime_support/templates/threadbridge-runtime-skill/SKILL.md](/Volumes/Data/Github/threadBridge/runtime_support/templates/threadbridge-runtime-skill/SKILL.md) and is installed under `.threadbridge/skills/threadbridge-runtime/` by the runtime bootstrap.

## Project Structure & Runtime Architecture
The runtime is organized in four layers:

- Desktop runtime owner and management plane: the macOS desktop entrypoint owns the local management API, tray/web management UI, runtime-owner reconcile loop, managed Codex preferences/builds, and machine-level runtime health authority.
- Shared runtime control and projection: internal services own workspace runtime ensure/repair, workspace session bind/new/repair, Telegram-to-live-TUI routing, and app-server observer projection. This layer is adapter-neutral and is where workspace control semantics now live.
- Telegram adapter: the Rust bot receives Telegram updates, enforces authorization, routes commands into shared runtime control, streams live Codex previews, and sends results back to Telegram, but it is no longer the formal runtime owner nor the primary home of runtime control orchestration.
- Tool executors: workspace-local wrapper commands under `.threadbridge/bin/` call Python scripts in `runtime_support/tools/` to materialize prompt configs, generated images, and Telegram outbox payloads.

Important repo areas:

- `rust/src/bin/threadbridge_desktop.rs`: desktop runtime entrypoint, tray host, and Telegram bot launcher.
- `rust/src/codex.rs`: app-server JSON-RPC client, thread lifecycle helpers, and event normalization for previews.
- `rust/src/management_api.rs`: local HTTP management API, workspace/thread views, control actions, and setup/runtime endpoints for the desktop management surface.
- `rust/src/runtime_owner.rs`: desktop runtime owner heartbeat, reconcile loop, and workspace runtime health authority.
- `rust/src/runtime_control.rs`: shared runtime control services for workspace runtime, session lifecycle, and Telegram/TUI routing.
- `rust/src/app_server_observer.rs`: app-server observer that projects preview/final/process events and emits adapter-neutral interaction events.
- `rust/src/runtime_interaction.rs`: shared interaction event types for `request_user_input`, resolved requests, and turn completion follow-up.
- `rust/src/process_transcript.rs`: normalized final/process transcript mapping shared by management UI and Telegram preview surfaces.
- `rust/src/workspace.rs`: workspace bootstrap logic that installs `.threadbridge/`, workspace-local runtime skills, wrappers, and runtime state surfaces.
- `rust/src/repository.rs`: persistent bot-local thread state for metadata, transcripts, session bindings, and image-state artifacts.
- `rust/src/thread_state.rs`: canonical thread state resolver for `lifecycle_status`, `binding_status`, and `run_status`.
- `rust/src/telegram_runtime/`: Telegram command handling, message flows, image handling, preview rendering, and adapter-owned interaction bridging.
- `runtime_support/templates/threadbridge-runtime-skill/`: workspace-local runtime skill template and references installed into `.threadbridge/skills/threadbridge-runtime/`.
- `runtime_support/tools/`: Python executors invoked from `.threadbridge/bin/*`.
- bot-local runtime data root: debug builds default to `data/`; bundled release builds default to the platform local app-data directory under `threadBridge/data`. In debug mode, `data/main-thread/` stores the control console state and each thread maps to `data/<thread-key>/`.

Treat `target/` and most bot-local runtime state as generated output.

## Runtime Instructions
There is one project `AGENTS.md` role and one threadBridge runtime skill role:

- Root `AGENTS.md`: this maintainer guide.
- `runtime_support/templates/threadbridge-runtime-skill/SKILL.md`: workspace-local threadBridge runtime skill installed under `.threadbridge/skills/threadbridge-runtime/`.

There is no thread-local runtime-root `<thread-key>/AGENTS.md` surface anymore.

Do not inject threadBridge runtime contracts into project `AGENTS.md` during ordinary workspace ensure, resume, or reconcile. Runtime capability documentation belongs in the workspace-local skill and its references. Legacy managed `AGENTS.md` cleanup belongs behind explicit runtime support rebuild/migration flows, not opportunistic reconcile.

## Workspace Lifecycle & Data Flow
The operational flow is: desktop runtime owner -> local management API / Telegram adapter -> Codex app-server thread -> real workspace runtime -> Python tool wrappers -> Telegram reply or local management surface.

From a maintainer perspective:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qoli/threadBridge](https://github.com/qoli/threadBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
