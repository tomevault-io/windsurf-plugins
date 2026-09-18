---
trigger: always_on
description: Hermes Desktop domain rules for Cursor. Defines product boundaries, domain vocabulary, module ownership, profile runtime rules, Web Operator rules, Windows deployment rules, and implementation guardrails.
---


# 005 - Hermes Desktop Domain Rules

## 1. Product Identity

This project is Hermes Desktop / Portal Desktop.

It is an Electron desktop shell for operating, configuring, observing, and extending a local or remote Python-based `hermes-agent`.

It is not the LLM runtime.
It is not the tool execution engine.
It is not the memory engine.
It is not the Hermes Gateway protocol implementation.

The desktop app owns:

- Electron window lifecycle
- Renderer UI
- Preload bridge
- IPC routing
- Local configuration UX
- Local filesystem orchestration
- Local process management
- Gateway lifecycle control
- Profile runtime control plane
- Web Operator desktop bridge
- Installer / bootstrap / diagnostics UI
- Windows desktop deployment experience

The Python `hermes-agent` owns:

- LLM inference routing
- Tool execution
- Memory retrieval
- Skill execution
- Gateway API behavior
- `/v1/chat/completions`
- agent-side reasoning and orchestration

Do not move Python backend responsibilities into Electron.

---

## 2. Core Process Boundary

The system has four runtime layers:

```text
Renderer Process
  React UI only
  No Node.js access
  Calls window.hermesAPI only

Preload Bridge
  Exposes typed hermesAPI
  Security boundary
  No business UI

Main Process
  Node.js privileged layer
  Owns IPC handlers
  Owns filesystem/process/SQLite/gateway lifecycle

Python Gateway
  External process
  Treated as black box
  Accessed through local HTTP/SSE or CLI fallback
```

Hard rules:

- Renderer must never import `electron`, `fs`, `path`, `child_process`, `better-sqlite3`, `os`, or Node-only modules.
- Renderer must never call `ipcRenderer` directly.
- Renderer must only use `window.hermesAPI`.
- Preload is the only bridge.
- Main Process is the only place for filesystem, SQLite, local process, Git, Python, NSIS, PATH, and installer operations.
- Python Gateway must be treated as an external service, not as in-process code.

---

## 3. Domain Vocabulary

Use these domain names consistently.

### App / Shell

- `Hermes Desktop`
- `Portal Desktop`
- `Desktop Shell`
- `Desktop Runtime`
- `Desktop Control Plane`

### Backend

- `hermes-agent`
- `Python Gateway`
- `Hermes Gateway`
- `Gateway Runtime`

### Profile

- `default profile`
- `specialist profile`
- `profile runtime`
- `profile home`
- `profile workspace`
- `profile gateway`
- `profile runtime db`

### Workspace

- `Portal Home`
- `Profile Workspace`
- `Runtime Center`
- `Web Operator`
- `Observability`
- `Local Install`
- `Settings`

### Automation

- `Desktop Tool Bridge`
- `BrowserController`
- `WebContentsView`
- `Browser IPC`
- `Web Operator Action`
- `Sensitive Action Confirmation`
- `DOM Snapshot`
- `Screenshot History`

### Windows Deployment

- `NSIS assisted installer`
- `Install Directory`
- `Runtime Root`
- `User PATH`
- `System PATH`
- `Bootstrap`
- `Local Doctor`
- `Install Log`

Do not invent alternative domain names unless explicitly requested.

---

## 4. Module Ownership

Respect the existing module responsibilities.

### `src/main/index.ts`

Single IPC registration hub.

Add new IPC handlers here only after implementing the domain logic in a separate `src/main/*.ts` module.

Pattern:

```ts
ipcMain.handle("domain:action", async (_, input) => {
  return domainAction(input);
});
```

Do not place large business logic directly inside `setupIPC()`.

---

### `src/preload/index.ts`

Authoritative renderer API surface.

Every Renderer capability must be declared here first.

Pattern:

```ts
const hermesAPI = {
  getRuntimeStatus: () =>
    ipcRenderer.invoke("runtime:get-status"),
};
```

Every long-lived event listener must return an unsubscribe function.

Pattern:

```ts
onInstallProgress: (callback) => {
  const listener = (_event, payload) => callback(payload);
  ipcRenderer.on("install-progress", listener);
  return () => ipcRenderer.removeListener("install-progress", listener);
};
```

---

### `src/preload/index.d.ts`

Runtime contract for the Renderer.

Every new `window.hermesAPI` method must have a matching TypeScript declaration.

Do not use `any`.
Use shared types from `src/shared/**`.

---

### `src/main/hermes.ts`

Gateway lifecycle module.

Allowed responsibilities:

- `startGateway`
- `stopGateway`
- `restartGateway`
- gateway health polling
- send message to Gateway
- SSE stream handling
- CLI fallback
- Gateway config injection before start

Do not put UI state here.
Do not put Renderer concerns here.
Do not modify profile runtime DB here unless explicitly part of gateway lifecycle.

---

### `src/main/installer.ts`

One-time installation and local environment bootstrap.

Allowed responsibilities:

- detect install status
- create Python venv
- install dependencies
- run doctor
- run update
- read installer logs
- manage bootstrap progress
- import/export backup

Do not put profile runtime governance here unless it belongs to initial bootstrap.

---

### `src/main/config.ts`

Profile-aware `.env` and `config.yaml` management.

Allowed responsibilities:

- model config
- provider config
- API key references
- local / remote mode
- platform toggles
- toolset toggles
- config cache


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
