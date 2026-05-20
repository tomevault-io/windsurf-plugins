---
trigger: always_on
description: - `iOS (App)/` holds the SwiftUI app code (`Features/`, `Shared/`, `Config/`, `App/`).
---

# Repository Guidelines

## Project Structure & Module Organization
- `iOS (App)/` holds the SwiftUI app code (`Features/`, `Shared/`, `Config/`, `App/`).
- `Shared (Extension)/Resources/` contains Safari extension assets: `manifest.json`, `content.js`, and `webllm/` popup HTML/JS/worker files.
- `Shared (Extension)/Resources/webllm-assets/` stores downloaded model/wasm files (gitignored).
- `Shared (Extension)/SafariWebExtensionHandler.swift` handles extension/native settings bridge.
- `Shared (App)/Assets.xcassets` holds shared images; `Docs/` and `Scripts/` contain guides and tooling; `assets/` is for marketing images.
- `eisonAI.xcodeproj` is the Xcode entry point.

## Build, Test, and Development Commands
- `python3 Scripts/download_webllm_assets.py` downloads model/wasm files into `Shared (Extension)/Resources/webllm-assets/` (required).
- `open eisonAI.xcodeproj` builds and runs the iOS/iPadOS app + Safari extension in Xcode.
- `Scripts/build_mlc_xcframeworks.sh` rebuilds MLC xcframeworks (expects `MLC_LLM_SOURCE_DIR`, outputs to `dist/`).

## Coding Style & Naming Conventions
- Swift uses 4-space indentation; SwiftUI views follow `FeatureNameView`/`FeatureNameViewModel` naming.
- Extension JavaScript uses 2-space indentation; file names are descriptive (`popup.js`, `contentReadability.js`).
- Keep constants uppercase with underscores (for example, `MODEL_ID`, `WASM_FILE`).
- Prefer small, single-purpose files under `Features/<FeatureName>/`.

## Testing Guidelines
- No automated test targets were found. Validate changes manually in Xcode and on real iOS/iPadOS devices (WebGPU behavior differs from the simulator).
- If adding tests, use XCTest with `*Tests.swift` and run via Xcode’s Test action or `xcodebuild test`.

## Commit & Pull Request Guidelines
- Commits follow Conventional Commits with a scope; emoji prefixes are common. Example: `✨ feat(library): 新增收藏功能`.
- Common types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, and `build` (seen in history).
- PRs should include a clear description, linked issues, and screenshots for UI/popup changes. Update `Docs/` when setup or behavior changes.

## Security & Configuration Notes
- Large assets in `Shared (Extension)/Resources/webllm-assets/` are intentionally gitignored; use the download script instead of committing binaries.
- Extension CSP and entry points live in `Shared (Extension)/Resources/manifest.json`; keep WebGPU/wasm settings aligned with WebLLM updates.

<!-- threadbridge:runtime:start -->
## threadBridge Runtime Appendix

This managed block is appended by threadBridge to a real project workspace `AGENTS.md`.

### Runtime Model

- The current working directory is the real bound workspace, not a projected copy.
- Preserve this workspace's own conventions and instructions. This appendix adds bot/runtime behavior; it does not replace project-local rules.
- threadBridge tracks Telegram-thread metadata outside the workspace under its own bot-local runtime data root. In debug builds this defaults to repo-local `data/`; in release builds it defaults to the platform local app-data directory. That bot-local state is not the source of truth for project files.
- Use the current Codex thread context as the primary continuity source. Do not rebuild long transcript replays unless a workflow explicitly requires it.

### Runtime Surface

- threadBridge installs wrapper commands under:
  - `./.threadbridge/bin/build_prompt_config`
  - `./.threadbridge/bin/generate_image`
  - `./.threadbridge/bin/hcodex`
  - `./.threadbridge/bin/send_telegram_media`
- threadBridge installs local shell/runtime sync files under:
  - `./.threadbridge/state/workspace-config.json`
  - `./.threadbridge/state/app-server/current.json`
  - `./.threadbridge/state/runtime-observer/current.json`
  - `./.threadbridge/state/runtime-observer/events.jsonl`
- threadBridge request/result files live under:
  - `./.threadbridge/tool_requests/`
  - `./.threadbridge/tool_results/`
- Keep these wrapper names and paths stable.
- `./.threadbridge/state/runtime-observer/*` is a workspace-local observation and activity surface.
- Treat desktop owner heartbeat and management/runtime protocol views as the canonical runtime-health authority, not `runtime-observer/*` by itself.

### Local Codex TUI

- Run `./.threadbridge/bin/hcodex` for the managed local TUI path in this workspace.
- `hcodex` resolves the shared workspace daemon from `./.threadbridge/state/app-server/current.json` and launches `codex --remote ...`.
- `hcodex` also reads `./.threadbridge/state/workspace-config.json` so local launch and resume use the workspace execution mode.
- With no extra args, `hcodex` starts a fresh local TUI session for this workspace.
- Fresh `hcodex` sessions project mirror activity from the existing live daemon stream; standalone observer attach is reserved for explicit resume flows.
- Use `hcodex resume <session-id>` when you explicitly want to continue an existing Codex session.

### `./.threadbridge/bin/build_prompt_config`

- Use this command when the current thread needs to build or refresh prompt artifacts in this workspace.
- Before running it, decide whether the current Codex thread already has enough information.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qoli/eisonAI](https://github.com/qoli/eisonAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
