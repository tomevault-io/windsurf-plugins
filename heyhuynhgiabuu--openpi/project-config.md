---
trigger: always_on
description: **Purpose:** Project-level operating rules for building OpenPi: a desktop workbench for the Pi coding agent (`@earendil-works/pi-coding-agent` v0.74.0+).
---

# OpenPi Project Rules

**Purpose:** Project-level operating rules for building OpenPi: a desktop workbench for the Pi coding agent (`@earendil-works/pi-coding-agent` v0.74.0+).
**Audience:** human developers and AI coding agents.
**Source references:** earendil-works/pi main repo (https://github.com/earendil-works/pi), especially `packages/coding-agent/docs/` (sdk.md, rpc.md, session-format.md, extensions.md) and `packages/agent/README.md`.

---

## Product Direction

OpenPi is a desktop workbench for the Pi coding agent. It wraps Pi's session tree, agent events, extensions, skills, and customizations in an Electron + SolidJS UI — not a terminal emulator clone, not a VS Code replacement.

Target UX: sessions sidebar (workspace-grouped, token/cost badges, filter/sort popover) + agent conversation (model selector, tool cards, queue controls) + customizations panel (modal with AI wizard, Extensions/Skills/Prompts/Themes/Packages) + persistent Git source control panel (Changes/Files tabs, commit workflow) + split-pane diff viewer + bottom terminal panel (Output tab + Terminal tab) + OpenCode-style command palette for commands, files, and sessions.

---

## Recommended Stack

| Layer | Choice |
|---|---|
| Shell | Electron + electron-vite + electron-builder |
| Renderer | SolidJS + TypeScript + Vite |
| Styling | Tailwind CSS + Kobalte/Radix-style primitives + Lucide Icons |
| State | Solid signals/memos plus Electron-main read models |
| Validation | Zod at every IPC/JSON boundary |
| Terminal | xterm.js + node-pty in Electron main |
| Diff renderer | @pierre/diffs (replaceable renderer only) |
| Pi integration | @earendil-works/pi-coding-agent SDK (imported in Electron main) |
| Persistence | SQLite via better-sqlite3 in Electron main |
| Secrets | OS keychain via Electron safeStorage |

---

## Current Beta Surface

Implemented slices currently include:
- Secure Electron main/preload boundary with typed IPC schemas and renderer-only UI authority.
- Pi session host integration with streaming conversation, model controls, steering/follow-up queue visibility, abort, fork, and session rename flows.
- Workspace/session sidebar with recent workspace restore, session search/sort/group controls, pinned/archive affordances, workspace hero metadata, and Git branch/last-modified summary.
- Customizations modal for Extensions, Skills, Prompts, Themes, Packages, Settings, General preferences, and Keybindings, including the `⇧⌘P` Command Palette binding.
- OpenCode-style command palette (`⇧⌘P`) searching commands, workspace files via `fff`, and historical sessions.
- Main-owned Git source control panel, file tree/search, file viewer, and split diff viewer; renderer never runs Git directly.
- Bottom terminal/output panel backed by main-owned PTY lifecycle.
- Dynamic app metadata exposed from Electron main for Welcome/customizations branding, OpenPi runtime icons, and tag-triggered beta CI/release workflows.

---

## Pi Integration Path

### SDK is primary (Pi's own recommendation for Node.js)

```typescript
// Electron main process
import {
  createAgentSession,
  SessionManager,
  AuthStorage,
  ModelRegistry,
  DefaultResourceLoader,
} from "@earendil-works/pi-coding-agent";

const { session } = await createAgentSession({
  cwd: workspacePath,
  sessionManager: SessionManager.create(workspacePath),
  authStorage: AuthStorage.create(),
  modelRegistry: ModelRegistry.create(authStorage),
});

session.subscribe((event: AgentSessionEvent) => {
  // Forward to renderer via IPC
  mainWindow.webContents.send("pi:event", event);
});

await session.prompt(text);
```

### RPC subprocess — only when process isolation is explicitly required

Pi's `--mode rpc` (strict JSONL over stdin/stdout) is the right choice when:
- process-level isolation is required (security constraint or separate process budget)
- integrating from a non-Node language

For OpenPi, start with SDK in Electron main. Switch to RPC subprocess only when a concrete reason requires it.

**RPC framing note:** Split records on `\n` only. Do not use Node `readline` — it also splits on Unicode separators inside JSON.

### Session replacement API

For new session, resume, fork, and clone flows, use `AgentSessionRuntime` — not `AgentSession` directly:

```typescript
import { createAgentSessionRuntime, createAgentSessionServices, createAgentSessionFromServices } from "@earendil-works/pi-coding-agent";

const runtime = await createAgentSessionRuntime(
  async ({ cwd, sessionManager, sessionStartEvent }) => {
    const services = await createAgentSessionServices({ cwd });
    return { ...await createAgentSessionFromServices({ services, sessionManager, sessionStartEvent }), services, diagnostics: services.diagnostics };
  },
  { cwd, agentDir: getAgentDir(), sessionManager: SessionManager.create(cwd) }
);

await runtime.newSession();   // replaces active session
await runtime.switchSession(path);
await runtime.fork(entryId);  // creates new session file
```

Re-subscribe to `AgentSessionEvent` after every session replacement — subscriptions attach to a specific `AgentSession` instance.

---

## Process Model and Authority

### Renderer is not authority


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heyhuynhgiabuu/openpi](https://github.com/heyhuynhgiabuu/openpi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
