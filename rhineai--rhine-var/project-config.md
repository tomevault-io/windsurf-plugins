---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rhine-Var is a CRDT-based collaborative state management library built on top of Yjs. It provides a proxy-based API inspired by Valtio that makes collaborative editing feel like working with regular JavaScript objects, with full TypeScript support and React integration.

## Build and Development Commands

### Setup

```bash
bun install                    # Install dependencies
bun run install-next          # Install Next.js playground dependencies
bun run link-next             # Link library to playground for testing
```

### Development

```bash
bun run dev                   # Watch mode - compile TypeScript on changes
bun run build                 # Build the library (outputs to dist/)
bun run playground            # Start Next.js playground (port 6700)
```

### Other

```bash
bun run commit                # Custom commit script (scripts/commit.js)
```

## Architecture

### Core Layers

The library is organized into distinct architectural layers:

**1. Native Layer (Yjs Wrapper)**

- `src/core/native/` - Wraps Yjs types (YMap, YArray, YText, YXmlElement, etc.)
- Provides type definitions and utilities for working with Yjs native objects
- The `Native` type represents any Yjs CRDT type

**2. RhineVar Layer (Base Classes)**

- `src/core/var/` - Core RhineVar classes that extend `RhineVarBase`
- `RhineVarBase` (src/core/var/rhine-var-base.class.ts) - Abstract base class providing:
  - Event subscription system (subscribe, subscribeKey, subscribeDeep, subscribeSynced)
  - Undo/redo via UndoManager
  - Awareness API for multi-user presence
  - JSON serialization (json(), frozenJson(), jsonString())
  - Parent-child hierarchy management
- Concrete implementations: `RhineVarMap`, `RhineVarArray`, `RhineVarText`, `RhineVarXmlElement`, etc.

**3. Proxy Layer (User-Facing API)**

- `src/core/proxy/rhine-proxy.ts` - Creates JavaScript Proxies around RhineVar objects
- `rhineProxy()` - Main entry point for creating collaborative state with server connection
- `rhineProxyGeneral()` - Just for create item inside
- Proxy handlers intercept get/set/delete operations and translate them to Yjs operations
- Support system (`src/core/var/support/`) adds array methods (push, pop, map, filter, etc.)

**4. Connector Layer (Network Sync)**

- `src/core/connector/` - Abstract connector system for syncing with servers
- `Connector` abstract class defines the interface
- `HocuspocusConnector` - Default implementation using @hocuspocus/provider
- `WebsocketConnector` - Alternative using y-websocket
- Manages YDoc lifecycle and sync state

**5. React Integration**

- `src/react/` - React hooks for using Rhine-Var in React apps
- `useRhine()` - Creates reactive snapshot that updates on changes
- `useSynced()` - Hook for sync status
- Separate export path: `rhine-var/react`

### Key Concepts

**Proxy Pattern**: Users interact with a JavaScript Proxy that looks like a normal object but internally operates on Yjs CRDTs. All mutations are automatically synced.

**Event System**: Three levels of subscriptions:

- `subscribe()` - Listen to direct property changes
- `subscribeKey()` - Listen to specific key changes
- `subscribeDeep()` - Listen to nested changes (bubbles up from children)
- `subscribeSynced()` - Listen to sync status changes

**Parent-Child Hierarchy**: RhineVar objects maintain parent references, allowing events to bubble up and enabling root-level features (connector, options, undoManager) to be accessed from any nested object.

**Native Access**: The `.native` property exposes the underlying Yjs object for advanced operations. Direct Yjs operations automatically trigger RhineVar updates.

## Path Aliases

The project uses TypeScript path aliases configured in tsconfig.json:

- `@/*` maps to `./src/*`

Always use these aliases when importing within the codebase (e.g., `import {foo} from "@/core/proxy/rhine-proxy"`).

## Testing with Playground

### 常规测试

在`playground/general`目录中，新建一个ts文件(内容可参考playground/general/task-common.ts)，作为测试文件。

并通过运行`bun run playground/general/task-xxx.ts`进行测试。查看代码是否符合预期。

服务器直接使用`ws://rvp.rhineai.com/task-xxx`。

### 前端测试

仅在明确是前端相关功能的时候使用 NextJs Playground 进行测试

The `playground/next/` directory contains a Next.js app for testing:

- Link the library first: `bun run link-next`
- Start dev server: `bun run playground`
- Example files in `playground/next/src/app/examples/`

## Important Implementation Notes

**Yjs Transaction Batching**: When making multiple changes, wrap them in a transaction for better performance:

```typescript
state.native.doc.transact(() => {
  // Multiple operations here
})
```

**Snapshot vs Proxy**: In React, `useRhine()` returns a read-only snapshot. Never mutate the snapshot - always mutate the original proxy object.

**Connector Creation**: When passing a string/number to `rhineProxy()`, it automatically creates a connector:

- Plain string/number → prepends default public URL (wss://rvp.rhineai.com/)
- Full URL → uses as-is
- Connector object → uses directly


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RhineAI/rhine-var](https://github.com/RhineAI/rhine-var) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
