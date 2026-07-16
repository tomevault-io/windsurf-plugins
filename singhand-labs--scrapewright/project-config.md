---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Scrapewright is an LLM-powered web scraping platform built as a Chrome Extension (Manifest V3) with a Node.js Native Messaging Host. External programs call configurable scraping services via a local HTTP API. The extension provides an AI-guided interactive wizard for service creation.

A **service** is a *step graph* (a small state machine of named steps), not a single script. Each step runs a scraping snippet in a sandbox; `onSuccess`/`onFailure` edges route control between steps to express loops, branches, and pagination. See "Step Graph model" below.

**Tech stack:** Chrome Extension MV3, Vanilla JS, Node.js >= 18. `node --test` (no Jest). Root `package.json` only provides `jsdom` (used by extension tests).

## Common Commands

```bash
# === Unified CLI (recommended entrypoint) — ./bin/scrapewright ===
# Auto-detects the extension id (no hand-copying), installs, diagnoses, restarts, tails logs.
./bin/scrapewright setup --auto    # one-shot install on a new machine
./bin/scrapewright status          # host process + /health + id-drift check
./bin/scrapewright doctor          # full diagnostic (exits non-zero if sick)
./bin/scrapewright restart         # kill host (then Reconnect in options to relaunch; use after editing host.js)
./bin/scrapewright logs -f         # tail host log
./bin/scrapewright id              # detect loaded extension id + drift vs manifest

# === Native host install / diagnose (macOS & Linux) — underlying primitives ===
cd native-host
./install-host.sh <extension-id>   # install (find id at chrome://extensions/)
./install-host.sh --doctor         # diagnostic: node, manifest, port, recent log
./install-host.sh --uninstall      # remove manifest + wrapper
node host.js --doctor              # host's own view: env, last 15 log lines

# === Native host install / diagnose (Windows, PowerShell) ===
cd native-host
.\install-host.ps1 <extension-id>  # install (find id at chrome://extensions/)
.\install-host.ps1 -Doctor         # diagnostic: node, manifest, port, registry, recent log
.\install-host.ps1 -Uninstall      # remove registry key + manifest + wrapper

# === Native host tests ===
cd native-host && npm test                       # node --test test/*.test.js
cd native-host && node --test test/host.test.js  # single file

# === Extension tests (need jsdom from root) ===
npm install                            # once, at repo root, for jsdom
cd extension && node --test test/*.test.js lib/*.test.js   # all
cd extension && node --test test/step-orchestrator.test.js # single file
# Note: tests live in BOTH extension/test/ and extension/lib/ (e.g. lib/service-registry.test.js).
# They require via relative paths ('../lib/...'), so run node --test from extension/.

# === Run host manually (dev mode; falls back to HTTP long-polling transport) ===
cd native-host && node host.js
node host.js --port=19880              # or SCRAPEWRIGHT_PORT=19880 node host.js
curl http://localhost:8765/health      # health check (no auth)

# === Inspect host log ===
tail -f ~/Library/Logs/scrapewright/host.log         # macOS
tail -f ~/.cache/scrapewright/host.log               # Linux
# Windows (PowerShell):
#   Get-Content -Wait "$env:LOCALAPPDATA\scrapewright\host.log" -Tail 20
# Override path with: SCRAPEWRIGHT_LOG_FILE=/path/to.log node host.js
# Boot crashes (before logger init) land in startup-error.log next to host.log.
```

After modifying extension files, reload the extension at `chrome://extensions/` (click the refresh icon on the extension card). Native Host changes require restarting Chrome or disconnecting/reconnecting the extension. The extension's options page (`options.html`) shows a **Native Host Status** card with the current connection mode (Connected / Polling / Disconnected), the last error, the log file path, a Reconnect button, and a Copy Diagnostics button — check there first when investigating connectivity issues.

`README.md` (English, default) and `README.zh-CN.md` (Chinese) are the canonical user-facing references — both document the full HTTP API, DSL, distributed deployment, and CDP comparison. `docs/technical-whitepaper.md` (中文) and `docs/technical-whitepaper.en.md` (English) are the design whitepaper — architecture, data flow, module reference, and extension guide. `req.md` is the original requirements spec (local only).

`scripts/` holds concrete reference service definitions (`baidu3.json`, `baidu4.json`, `yuanbao.json`) — full persisted service objects including `steps`, `inputSchema`, `outputSchema`, and `config`. These are the best examples of the step-graph DSL in the wild; read one before authoring or debugging a service.

## Native Host install internals

`install-host.sh` (macOS/Linux) and `install-host.ps1` (Windows) each generate two artifacts:

1. **A wrapper script** that `exec`s node via its absolute path:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [singhand-labs/scrapewright](https://github.com/singhand-labs/scrapewright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
