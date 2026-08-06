---
trigger: always_on
description: 1. Fail fast. Never hide errors behind fallback behavior or pretend a failed operation succeeded.
---

# Pi engineering contract

## Non-negotiable engineering rules

1. Fail fast. Never hide errors behind fallback behavior or pretend a failed operation succeeded.
2. Fix root causes. Do not accumulate one-off patches around a defect.
3. Make failures observable. Critical host, RPC, persistence, and renderer failures must leave useful structured logs.
4. Design for traceability. Important process starts, protocol failures, state transitions, and destructive operations must be diagnosable.
5. Keep this file current. Update it in the same change whenever the product direction, runtime architecture, or critical workflow changes.
6. Protect the mainline. Create a dedicated branch before broad refactors or experimental work.

## Product direction

- This repository continues the Git history and GitHub location of Ousia, but the maintained product is now the standalone, lightweight Tauri application Pi. The final Electron state is preserved on `codex/archive-ousia-electron-v0.1.32`; Ousia is otherwise an upstream UI/UX reference only and must never be modified as part of Pi work.
- The user-facing product and packaged application name is `Pi`. Keep the internal `pi-gui` crate/package names, bundle identifier, data directories, log filename, environment variables, and existing PATH ownership marker stable for upgrade compatibility.
- Preserve Ousia's UI, interactions, copy, spacing, typography, and behavior except for intentional Pi deviations documented in this section. Limit other frontend changes to build fixes and the minimum host/runtime adaptation required by Tauri.
- Pi intentionally uses the system UI font only. Do not expose font-family settings or bundle alternate font files.
- The only agent harness is Pi. There is no Codex harness, provider switch, compatibility layer, or alternate-agent implementation.
- Messages sent while Pi is already running default to the follow-up queue. Users may explicitly switch the conversation behavior to steering. The queue floats above the conversation as a composer-owned overlay; queue growth must never resize the conversation viewport or change its scroll-follow state. The conversation bottom clearance must include the queue's measured overlap beyond its existing bottom padding so followed content remains actually visible rather than merely reaching the mathematical scroll maximum.
- Do not bundle Node.js, Pi, or a Pi SDK in the application package. Resolve and launch an external `pi` executable so the app shares the user's Pi configuration, credentials, models, extensions, and sessions.
- The application may optionally install Pi with the user's existing Node.js/npm into an application-owned prefix outside the `.app`. This managed installation must never modify the system npm prefix and uninstall must never remove the user's `~/.pi` data.
- A missing Pi executable is an on-demand onboarding state, not a startup chat error. Keep the empty chat neutral; when the user first tries to send, retain the draft and open the Pi install/select dialog.
- Pi configuration and credentials are read-only inputs. Do not add API-key mutation commands or create a second credential/configuration source in the application.
- Non-project sessions and the project folder picker default to `~/pi`. Non-project sessions snapshot their working directory when created, so later default changes cannot invalidate persisted Pi mappings. Pi's previous home-directory default and Ousia's historical `~/Documents/Ousia` and `~/.ousia/chat` defaults are migrated to `~/pi` on load and persisted before the Rust host resolves a chat context; existing sessions retain their original directory while new sessions use the migrated default.

## Architecture source of truth

- UI: React 19 + TypeScript + Vite, mirrored from Ousia under `src/`.
- Desktop host: Tauri 2 + Rust under `src-tauri/`.
- Compatibility boundary: `src/tauri/api.ts` implements the existing `window.ousia` UI contract with Tauri commands and events. Keeping the contract avoids UI/UX drift; it is not an Electron runtime.
- Agent boundary: the Rust host launches `pi --mode rpc` and exchanges strict line-delimited JSON over stdin/stdout.
- Pi discovery: `PI_GUI_PI_PATH` is the authoritative development/testing override; otherwise prefer a persisted explicitly selected or application-managed executable, then inspect the login-shell `PATH`, common installation locations, and the active npm global prefix. Every selected path must be a verified executable.
- Runtime ownership: `pi-runtime.json` in the Tauri application data directory is the source of truth for a managed Pi installation and optional shell integration. PATH integration may only own `~/.local/bin/pi` plus one exact marked block in `~/.zprofile` or `~/.bash_profile`; removal must validate the receipt before changing either file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s1dashu/ousia](https://github.com/s1dashu/ousia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
