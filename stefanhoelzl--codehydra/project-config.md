---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

---

## CRITICAL RULES

These rules MUST be followed. Violations require explicit user approval.

### No Ignore Comments

**NEVER add without explicit user approval:**

- `// @ts-ignore`, `// @ts-expect-error`, `// eslint-disable*`, `any` type assertions
- Modifications to `.eslintignore`, `.prettierignore`

### API/IPC Interface Changes

**NEVER modify without explicit user approval:**

- IPC channel names/signatures (`api:project:*`, `api:workspace:*`)
- Intent/event type definitions, operation interfaces
- Preload script exposed APIs, event names/payloads, shared types in `src/shared/`

**Why**: IPC contracts affect main/renderer sync, type safety, and backwards compatibility.

### New Boundary Interfaces

**NEVER add without explicit user approval:**

- New abstraction interfaces (`*Layer`, `*Client`, `*Provider`)
- New boundary types (I/O, network, filesystem, process abstractions)
- Entries to External System Access Rules table

**Why**: Architectural decisions with maintenance burden. Must follow established patterns.

### External System Access Rules

All external access MUST use abstraction interfaces:

| External System  | Required Interface                    | Forbidden Direct Access |
| ---------------- | ------------------------------------- | ----------------------- |
| Filesystem       | `FileSystemBoundary`                  | `node:fs/promises`      |
| HTTP requests    | `HttpClient`                          | `fetch()`               |
| Port operations  | `PortManager`                         | `net` module            |
| Process spawning | `ProcessRunner`                       | `execa`                 |
| Agent operations | `AgentProvider`, `AgentServerManager` | Direct OpenCode SDK     |
| OpenCode API     | `SdkClientFactory`                    | Direct HTTP/SSE         |
| Git operations   | `IGitClient`                          | `simple-git`            |
| Electron Window  | `WindowBoundary`                      | `BaseWindow`            |
| Electron View    | `ViewBoundary`                        | `WebContentsView`       |
| Electron Session | `SessionBoundary`                     | `session`               |
| Electron IPC     | `IpcBoundary`                         | `ipcMain`               |
| Electron Dialog  | `DialogBoundary`                      | `dialog`                |
| Electron Image   | `ImageBoundary`                       | `nativeImage`           |
| Electron App     | `AppBoundary`                         | `app`                   |
| Electron Menu    | `MenuBoundary`                        | `Menu`                  |

**Acceptable exceptions**: Third-party libraries that encapsulate their own I/O (like `ignore`, `posthog-node`) do not need abstraction layers. We abstract our own I/O, not the internals of external libraries.

### Path Handling

**ALWAYS use the `Path` class** for internal path handling:

```typescript
import { Path } from "../services/platform/path";
const projectPath = new Path(inputPath);
map.set(path.toString(), value); // toString() for Map keys
path1.equals(path2); // equals() for comparison
```

**Rules**: Services receive `Path` objects. IPC uses strings. Convert at IPC boundary.

### Network

**ALWAYS use `127.0.0.1`** instead of `localhost` for local connections.

### Ask When Uncertain

**NEVER make decisions based on assumptions.** If multiple plausible causes exist or you cannot verify an issue, ask before proceeding.

---

## Documented Exceptions

Some components use external libraries directly without abstraction layers. These are approved exceptions where abstraction provides no benefit.

| Component       | Direct Dependency  | Reason                                                                                            |
| --------------- | ------------------ | ------------------------------------------------------------------------------------------------- |
| `AutoUpdater`   | `electron-updater` | Singleton with Electron lifecycle integration; no meaningful abstraction or isolated test benefit |
| `Config.load()` | `node:fs`          | Config must load synchronously before Electron app.ready; FileSystemBoundary is async-only        |

---

## Quick Reference

### Tech Stack

| Layer           | Technology                               |
| --------------- | ---------------------------------------- |
| Desktop         | Electron (BaseWindow + WebContentsViews) |
| Frontend        | Svelte 5 + TypeScript + @vscode-elements |
| Backend         | Node.js services                         |
| Testing         | Vitest                                   |
| Build           | Vite                                     |
| Package Manager | pnpm                                     |

### Essential Commands

| Command             | Purpose                                                                    |
| ------------------- | -------------------------------------------------------------------------- |
| `pnpm dev`          | Start development mode                                                     |
| `pnpm validate:fix` | Fix lint/format issues, run tests                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stefanhoelzl/codehydra](https://github.com/stefanhoelzl/codehydra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
