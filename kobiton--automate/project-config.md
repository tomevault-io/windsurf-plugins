---
trigger: always_on
description: Cross-tool agent instructions for the Kobiton mobile testing platform's MCP plugin. This file is the host-agnostic equivalent of `skills/run-automation-suite/SKILL.md`. It's read by Gemini CLI (via `contextFileName`), GitHub Copilot CLI, and Cursor CLI. Codex CLI reads the mirrored `.codex/skills/*/SKILL.md` instead; Claude Code reads `skills/*/SKILL.md` directly.
---

# Kobiton Automate - Agent Guide

Cross-tool agent instructions for the Kobiton mobile testing platform's MCP plugin. This file is the host-agnostic equivalent of `skills/run-automation-suite/SKILL.md`. It's read by Gemini CLI (via `contextFileName`), GitHub Copilot CLI, and Cursor CLI. Codex CLI reads the mirrored `.codex/skills/*/SKILL.md` instead; Claude Code reads `skills/*/SKILL.md` directly.

## What this plugin does

Kobiton is a real-device mobile cloud for Android + iOS testing. This MCP plugin gives AI agents tools to:

- **Devices**: list, get status, reserve, terminate reservation
- **Apps**: list, upload, confirm upload, get parsing status, get details
- **Sessions**: list, get, get artifacts, get user-input events, terminate
- **Test management**: create / list / get / update / delete test cases, test runs, and test suites; `saveTestCase` converts a finished manual session into a reusable test case
- **Setup**: `getCredential` (used by `/automate:setup`)

The MCP server runs at `https://api.kobiton.com/mcp`. Authentication is OAuth 2.1 (default) or API key (CI/headless).

## userIntent format

Every tool call requires a `userIntent` argument summarizing what the user is trying to accomplish, a natural-language sentence is sufficient (e.g., `"reserve a Pixel 7 to run the checkout suite"`). The plugin's audit logging consumes this; include it on every tool call.

## When the user asks to run tests on Kobiton

Default workflow (mirrors the `run-automation-suite` skill):

1. **Identify the app**: ask the user whether to upload a new app build or reuse an existing one. Do NOT auto-upload without confirmation. After `confirmAppUpload`, poll `getAppParsingStatus(versionId)` until the state is terminal (`OK` or a `FAILURE_*` value). See Known limitations.
2. **Select a device**: call `listDevices` with the right platform filter. Confirm with the user before reserving.
3. **Parse capabilities**: read the local Appium test script (Node / Python / .NET / Java), extract the capabilities literal, reconcile against the selected device per the must-match / suggested-default / user-controlled policy in `skills/run-automation-suite/references/capabilities.md`.
4. **Confirm and execute**: present the summary, get user confirmation, run the script in the background, open the live-view URL.
5. **Collect artifacts**: after the session terminates, call `getSession` + `getSessionArtifacts` for video, logs, screenshots, test reports. Surface session link + pass/fail.

Detailed step-by-step instructions live in `skills/run-automation-suite/SKILL.md`. Hosts that support skills load it automatically; otherwise read the file directly for the full workflow.

## When the user asks to interactively drive a device

For exploratory testing or repro work (not running a pre-written script):

1. **Pick a device**: same `listDevices` flow as above; the user is interactively in the loop.
2. **Create or resume a session**: `reserveDevice` then start an interactive session; resume an existing one by session ID if the user has one.
3. **Interact**: relay WebDriver commands through the plugin; capture artifacts on demand.
4. **End the session**: `terminateSession` when the user is done.

Detailed step-by-step instructions live in `skills/run-interactive-cli-session/SKILL.md`. Response shapes for the WebDriver layer are documented at `skills/run-interactive-cli-session/references/response-shapes.md`.

## When the user asks to drive a device from a natural-language intent

**Pick this skill** for agent-driven flows the user describes in plain language ("open YouTube and play the first world cup video", "log in then enable Bluetooth, then go home") — it auto-pilots from observation to action without a human in the loop on each step, and the result is a saveable test case. It complements (does NOT replace) `run-interactive-cli-session`: that one is for human-driven exploration via the CLI session type; this one uses the automation session type via direct Appium HTTP. (Tool names below are the Kobiton MCP tools' bare names — the host resolves the registered prefix.)

1. **Ask before picking the device and the live view** (the skill blocks here): which device + which observation mode (foreground live view vs background run). For the device, the same `listDevices` / `reserveDevice` flow as the other skills applies.
2. **Render capabilities** via `skills/run-automation-suite/scripts/render-capabilities.js` with `--newCommandTimeout 1800` (30 min — survives human-in-the-loop pauses) and `--scriptlessCapture` (so the resulting session is consumable by `saveTestCase`).
3. **Create the automation Appium session** via the Node-only `skills/drive-automation-session/scripts/appium.js` (no package deps — uses `node:https` directly). The script reads `~/.kobiton/.credentials` (written by `/automate:setup`) directly on each invocation — credentials never pass through argv, env, or the host transcript. Returns the session ID.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kobiton/automate](https://github.com/kobiton/automate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
