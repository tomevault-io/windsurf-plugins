---
trigger: always_on
description: This file is the durable repository-wide working contract for AI coding agents.
---

# AGENTS.md — EVREN Codex Bridge / EVREN Codex Desktop

This file is the durable repository-wide working contract for AI coding agents.

Read this file before making changes.

---

## 1. Project purpose

EVREN Codex Bridge started as a local compatibility bridge between OpenAI Codex CLI and the EVREN Responses API.

The project is evolving into **EVREN Codex Desktop**, a Windows desktop coding-agent application built around:

- Electron
- React
- TypeScript
- Vite
- the existing EVREN Bridge core
- OpenAI Codex App Server
- EVREN `/v1/responses`

Target normal-user flow:

```text
EVREN Codex Desktop
→ enter EVREN API key
→ select model
→ open project
→ chat with Codex
→ close app
→ return later
→ Continue Work
```

Normal users should not need to manually start the Bridge or open a second Codex terminal.

---

## 2. Current development state

The v2 Desktop implementation is under active development.

Do not treat v2 as publicly released unless the user explicitly requests a release.

Current Codex compatibility target:

```text
Codex CLI 0.157.1
upstream tag: rust-v0.157.1
```

Do not silently upgrade this target.

For Codex protocol work, inspect the exact upstream 0.157.1 schema/source before using methods, notifications, params, or response fields.

---

## 3. Core architecture

Normal Desktop flow:

```text
React Renderer
→ typed preload IPC
→ Electron Main
→ Codex App Server
→ local authenticated Bridge
→ EVREN Responses API
```

Electron Main owns privileged operations.

The renderer must remain unprivileged.

Reuse the existing Bridge runtime instead of duplicating it.

---

## 4. Electron security invariants

Do not weaken these without explicit user approval:

```text
nodeIntegration = false
contextIsolation = true
sandbox = true
webSecurity = true
```

The renderer must NOT receive direct access to:

- `fs`
- `child_process`
- `process.env`
- raw `ipcRenderer`
- raw JSON-RPC transport
- credentials
- arbitrary shell execution

Use a narrow typed preload API.

Production preload must remain Electron-sandbox compatible.

Expected production preload:

```text
dist/desktop/preload/index.cjs
```

Do not regress to an ESM preload that Electron cannot load.

If preload is unavailable, preserve the visible safe failure state rather than rendering a blank window.

---

## 5. EVREN API key rules

The EVREN API key is sensitive.

It may exist:

- temporarily in renderer while the user types it
- in Electron Main memory
- encrypted through Electron `safeStorage`

It must NEVER be:

- committed
- logged
- returned to renderer after storage
- written plaintext to settings
- written to `.env`
- placed in localStorage/sessionStorage
- included in diagnostics
- passed to the Codex child process

There must be no plaintext persistence fallback if secure storage is unavailable.

Session-only use is acceptable.

---

## 6. Codex → Bridge authentication

Desktop Codex must not receive the real EVREN API key.

Each application launch generates a random local Bridge credential.

Conceptually:

```text
Codex
→ per-launch local bearer token
→ local Bridge
→ EVREN API key owned by Electron Main
→ EVREN
```

The local Bridge token must:

- never be shown
- never be persisted
- never be logged
- never be forwarded upstream to EVREN

The Desktop Bridge must bind to:

```text
127.0.0.1
```

Desktop should use an ephemeral port.

---

## 7. Codex configuration

Desktop must not modify the user's global:

```text
~/.codex/config.toml
```

Use process-level Codex configuration overrides.

Production must not require:

- ChatGPT login
- OpenAI login
- OpenAI API key

The EVREN Desktop custom provider should remain configured as not requiring OpenAI authentication.

---

## 8. Codex App Server

Desktop communicates with Codex through App Server JSON-RPC.

Do NOT parse Codex TUI or ANSI output.

Current important methods include:

```text
initialize
thread/start
thread/list
thread/resume
thread/archive
thread/name/set
thread/turns/list
thread/items/list
turn/start
turn/interrupt
```

Relevant notifications include:

```text
thread/started
thread/status/changed
turn/started
turn/completed
turn/diff/updated
turn/plan/updated
item/started
item/completed
item/agentMessage/delta
item/commandExecution/outputDelta
item/fileChange/outputDelta
item/fileChange/patchUpdated
thread/tokenUsage/updated
thread/compacted
warning
error
```

Do not invent protocol shapes.

Unknown server requests must never be auto-approved.

---

## 9. Approval safety

Existing Desktop coding posture should remain safe.

Default behavior should stay equivalent to:

```text
approvalPolicy = on-request
approvalsReviewer = user
sandbox = workspace-write
```

Do not default to unrestricted execution.

Command/file approval decisions must map only to valid Codex protocol decisions.

Do not auto-approve unknown or malformed requests.

Cross-thread approval isolation must remain intact.

---

## 10. Never expose raw reasoning

Never expose, persist, log, or send raw chain-of-thought / hidden reasoning to the renderer.

The UI may show generic states such as:

```text
Thinking...
Working...
```

but not hidden reasoning content.

Renderer-safe DTO/projector code must exclude reasoning payloads.

---

## 11. Codex thread authority


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [berkaycari/evren-codex-proxy](https://github.com/berkaycari/evren-codex-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
