---
trigger: always_on
description: **Generated:** 2026-02-06
---

# kkrpc - PROJECT KNOWLEDGE BASE

**Generated:** 2026-02-06
**Commit:** (current)
**Branch:** main

## OVERVIEW

TypeScript-first RPC library with bidirectional communication across Node.js, Deno, Bun, Browser, and Tauri. Supports 15+ transport protocols with full type safety and zero-copy transferable objects. Includes language interop for Go, Python, Rust, and Swift.

## STRUCTURE

```
kkrpc/
├── packages/kkrpc/           # Core library
│   ├── src/                  # Source code
│   │   ├── channel.ts         # RPCChannel core
│   │   ├── interface.ts       # IoInterface abstraction
│   │   ├── adapters/         # Transport adapters (22 adapters)
│   │   ├── transfer*.ts       # Transferable objects support
│   │   └── serialization.ts  # JSON/superjson serialization
│   ├── __tests__/            # Bun test suite (17+ tests)
│   ├── __deno_tests__/       # Deno regression tests
│   ├── mod.ts                # Main entry (Node/Deno/Bun)
│   ├── browser-mod.ts        # Browser entry
│   └── dist/                 # Build output (do not edit)
├── packages/demo-api/         # Sample API implementation
├── packages/slidev/           # Presentation slides
├── examples/                  # 10+ usage examples
├── interop/                   # Language interop (Go, Python, Rust, Swift)
├── docs/                      # Documentation site
└── package.json               # pnpm workspace config
```

## WHERE TO LOOK

| Task                | Location                           | Notes                                      |
| ------------------- | ---------------------------------- | ------------------------------------------ |
| Core implementation | `packages/kkrpc/src/`              | channel.ts, interface.ts, serialization.ts |
| Transport adapters  | `packages/kkrpc/src/adapters/`     | 22 transport protocol adapters             |
| Validation          | `packages/kkrpc/src/validation.ts` | Standard Schema runtime validation         |
| Test code           | `packages/kkrpc/__tests__/`        | Bun tests, covers all adapters             |
| Deno compatibility  | `packages/kkrpc/__deno_tests__/`   | Deno regression tests                      |
| Usage examples      | `examples/`                        | HTTP, WebSocket, Worker, Chrome Extension  |
| AI skills           | `skills/`                          | Claude Code SKILL.md files                 |
| Build config        | `turbo.json`, `tsdown.config.ts`   | Turbo + tsdown build system                |

## CODE MAP

| Symbol                        | Type      | Location                          | Role                                  |
| ----------------------------- | --------- | --------------------------------- | ------------------------------------- |
| RPCChannel                    | Class     | src/channel.ts                    | Bidirectional RPC channel core        |
| IoInterface                   | Interface | src/interface.ts                  | Transport layer abstraction interface |
| IoCapabilities                | Interface | src/interface.ts                  | Adapter capability declarations       |
| serialize/deserialize         | Function  | src/serialization.ts              | Message serialization                 |
| transfer()                    | Function  | src/transfer.ts                   | Mark zero-copy objects                |
| RPCValidationError            | Class     | src/validation.ts                 | Validation error with context         |
| defineMethod()                | Function  | src/validation.ts                 | Schema-first method definition        |
| defineAPI()                   | Function  | src/validation.ts                 | Schema-first API definition           |
| extractValidators()           | Function  | src/validation.ts                 | Extract validators from defined API   |
| NodeIo                        | Class     | adapters/node.ts                  | Node.js stdio                         |
| DenoIo                        | Class     | adapters/deno.ts                  | Deno stdio                            |
| BunIo                         | Class     | adapters/bun.ts                   | Bun stdio                             |
| WorkerParentIO                | Class     | adapters/worker.ts                | Web Worker parent side                |
| WorkerChildIO                 | Class     | adapters/worker.ts                | Web Worker child side                 |
| TauriShellStdio               | Class     | adapters/tauri.ts                 | Tauri shell plugin adapter            |
| ElectronIpcMainIO             | Class     | adapters/electron-ipc-main.ts     | Electron main IPC                     |
| ElectronIpcRendererIO         | Class     | adapters/electron-ipc-renderer.ts | Electron renderer IPC                 |
| ElectronUtilityProcessIO      | Class     | adapters/electron.ts              | Electron utility process (main)       |
| ElectronUtilityProcessChildIO | Class     | adapters/electron-child.ts        | Electron utility process (child)      |

## CONVENTIONS

### Code Style

- **File naming**: TypeScript files use kebab-case (e.g. `stdio-rpc.ts`)
- **Export naming**: Classes/interfaces use PascalCase (`RPCChannel`), functions use camelCase (`generateUUID`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunkunsh/kkrpc](https://github.com/kunkunsh/kkrpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
