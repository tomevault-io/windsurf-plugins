---
trigger: always_on
description: Picky is a local-first macOS command center for Pi sessions. It captures neutral desktop context, sends it to local Pi through `picky-agentd`, and shows long-running Pickles in the Picky dock. Picky should stay thin: context capture, overlay/session UI, and session control. Pi remains responsible for interpreting intent, choosing skills/tools/MCPs, and doing the work.
---

# AGENTS.md - Picky Maintenance Guide

## Product intent

Picky is a local-first macOS command center for Pi sessions. It captures neutral desktop context, sends it to local Pi through `picky-agentd`, and shows long-running Pickles in the Picky dock. Picky should stay thin: context capture, overlay/session UI, and session control. Pi remains responsible for interpreting intent, choosing skills/tools/MCPs, and doing the work.

## Non-negotiable architecture rules

- Keep local-first behavior. No SaaS backend, auth, billing, remote analytics, or remote STT/TTS requirement for v1.

- Preserve long-running Pickle UX: multiple sessions, states, tool activity, logs, follow-up, abort, completion notification, artifacts, persistence/reconnect.

- Do not restart the running Picky app unless the user explicitly asks.

- Do not change Xcode defaults to always sign. Use `./scripts/package-signed-app.sh` only when a signed app bundle is needed.

## Current architecture

```text
Picky.app (SwiftUI/AppKit)
  -> WebSocket local protocol
picky-agentd (Node/TypeScript)
  -> Pi SDK runtime
local ~/.pi/agent skills/extensions/MCP/tools
```

Default daemon port is `127.0.0.1:17631`. Mock runtime is available via `PICKY_AGENTD_RUNTIME=mock`.

Packaged Picky.app bundles a pinned Node 22.x arm64 runtime under `Contents/Resources/agentd-runtime/bin/node`, signed separately with `Picky/NodeRuntime.entitlements` for V8 JIT. The launcher (`Picky/PickyAgentDaemonLauncher.swift`) resolves Node in this order:

1. `PICKY_NODE_PATH` env override (dev/debug).
2. Bundled `Resources/agentd-runtime/bin/node`.
3. `/usr/bin/env node` from inherited PATH (dev builds, `PICKY_SKIP_NODE_BUNDLE=1` packages).

Node version is single-sourced from `agentd/package.json#engines.node` (exact pin, no range). `scripts/fetch-node-runtime.sh` downloads + SHA256-verifies + caches under `build/cache/node/`. `agentd.node-preflight.json` records which source the launcher chose.

## Distribution identity

The upstream appcast URL, bundle identifier, logging subsystem, and keychain service currently use the maintainer's personal namespace (`Jonghakseo` / `com.jonghakseo.picky`). Forks or downstream distributions must replace those identifiers, Sparkle appcast URL, signing settings, and feedback Slack configuration with their own values before shipping.

## Optional Pi handoff command

Picky writes a local capability file for Pi extensions while `picky-agentd` is running:

```text
~/Library/Application Support/Picky/agentd-connection.json
```

For local development, enable the bundled handoff command by symlinking it into the local Pi extensions directory:

```bash
mkdir -p ~/.pi/agent/extensions
ln -sfn "$PWD/pi-extensions/picky-handoff" ~/.pi/agent/extensions/picky-handoff
```

After restarting Pi or running `/reload`, use:

```text
/handoff-to-picky continue this investigation in Picky and produce a final report
```

If Pi is mid-turn, the command first aborts the current turn and waits for it to settle. It then creates a new visible Pickle in Picky seeded with the current Pi session file, cwd, and recent branch excerpt as neutral context, and sends the kickoff instruction (defaults to `continue` when no argument is given) as the first user message so the Pickle resumes the work automatically.

## Code navigation index

When the user asks about a feature, start here before broad searching:

- App lifecycle / menu bar / permissions: `Picky/PickyApp.swift`, `Picky/App/`, `Picky/Companion/CompanionPanel*.swift`
- Settings / default cwd / local paths: `Picky/App/Settings/`, `Picky/App/Settings/PickySettingsStore.swift`
- Voice / push-to-talk / dictation: `Picky/CompanionManager.swift`, `Picky/BuddyDictationManager.swift`, `Picky/Companion/Dictation/`
- Global shortcut semantics/settings: `Picky/Shortcuts/`, `Picky/Companion/Dictation/GlobalPushToTalkShortcutMonitor.swift`, `Picky/Companion/Dictation/BuddyPushToTalkShortcut.swift`, `Picky/QuickInput/QuickInputDoubleTapDetector.swift`
- Quick text input: `Picky/QuickInput/`
- Speech transcription/playback providers: `Picky/Companion/Dictation/AppleSpeechTranscriptionProvider.swift`, `Picky/Companion/Dictation/BuddyTranscriptionProvider.swift`, `Picky/Companion/AzureOpenAI/`, `Picky/Companion/ElevenLabs/`, `Picky/Companion/Speech/`
- OpenAI Realtime voice mode (opt-in): `Picky/Companion/Realtime/`, `agentd/src/runtime/openai-realtime-main-runtime.ts`, `agentd/src/runtime/selectable-main-runtime.ts`, runtime selection in `agentd/src/bootstrap.ts`
- Screen/context capture: `Picky/Context/`, `Picky/PickyAdvancedContext.swift`, `Picky/Context/PickyContextPacketAssembler.swift`
- HUD shell / dock / Pickle container: `Picky/HUD/`, `Picky/HUD/PickyHUDView.swift`, `Picky/PickySessionViewModel.swift`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jonghakseo/picky](https://github.com/Jonghakseo/picky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
