---
trigger: always_on
description: General-purpose authentication CLI with pluggable strategies and browser adapters.
---

# SigCLI

General-purpose authentication CLI with pluggable strategies and browser adapters.
TypeScript, ES2022, strict mode, ESM (`"type": "module"`), Node >= 18.

When you make this project, do not add you as an author in commit. e.g. DO NOT add this in commit message "Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"

## Architecture

```
bin/sig.js (entry)  ──▶  cli/main.ts (router)  ──▶  cli/commands/*
                                                             │
deps.ts (composition root) ── wires all deps via DI ────────▶ AuthManager ──▶ Strategies + Storage + Browser
                                                             │
core/ (types, interfaces, Result, errors) ── zero external deps, imported by all layers
```

- **`bin/sig.js`** — Entry point. Auto-builds if needed, delegates to CLI router.
- **`src/deps.ts`** — Composition root. Creates registries, storage, browser factory, AuthManager. No singletons. Shared by CLI and programmatic API.
- **`src/auth-manager.ts`** — Orchestrator. Flow: stored cred → validate → refresh → authenticate. All methods return `Result<T, AuthError>`.
- **`src/core/`** — Shared vocabulary. Zero external dependencies.
- **`src/cli/`** — CLI commands (init, doctor, get, login, request, status, logout, providers, remote, sync, watch, rename, remove, completion, proxy). Each command is a standalone module. `init`, `doctor`, and `completion` run without deps (before config exists).
- **`src/strategies/`** — Each strategy: private class + exported `*StrategyFactory` (IAuthStrategyFactory).
- **`src/browser/adapters/`** — Browser automation. PlaywrightAdapter is the reference. Three-class pattern: Adapter → Session → Page.
- **`src/browser/flows/`** — `runHybridFlow` (headless→CDP→visible cascade), `runCdpFlow` (native browser CDP), `extractOAuthTokens`, `isLoginPage`.
- **`src/browser/detect-native.ts`** — Native browser binary detection (Chrome/Edge/Chromium) across macOS/Windows/Linux.
- **`src/browser/cdp-ws.ts`** — Minimal raw WebSocket CDP client (Node 18 compatible, no external deps).
- **`src/storage/`** — DirectoryStorage (per-file JSON + file lock + AES-256-GCM encryption), CachedStorage (TTL decorator), MemoryStorage (tests).
- **`src/crypto/`** — Encryption at rest. AES-256-GCM encrypt/decrypt, key generation/loading. Key stored at `~/.sig/encryption.key`.
- **`src/providers/`** — ProviderRegistry (URL→provider via domain matching), config-loader (YAML/JSON).
- **`src/sync/`** — SyncEngine + SshTransport for credential sync to remote machines. Encrypts with per-remote key. RemoteConfig in `~/.sig/config.yaml`.
- **`src/proxy/`** — MITM proxy daemon. CaManager (ECDSA P-256 CA + per-hostname leaf certs), ProxyServer (HTTP/HTTPS CONNECT with credential injection), daemon (proxy + watch loop), proxy-state (PID/port files at `~/.sig/proxy/`).
- **`src/utils/`** — JWT decode, duration parse, HTTP helpers.

## Key Interfaces

| Interface              | Location                                 | Methods                                                 | Extend when                |
| ---------------------- | ---------------------------------------- | ------------------------------------------------------- | -------------------------- |
| `IAuthStrategy`        | `src/core/interfaces/auth-strategy.ts`   | `validate`, `authenticate`, `refresh`, `applyToRequest` | Adding a new auth method   |
| `IAuthStrategyFactory` | same file                                | `name`, `create(config)`                                | Wrapping a new strategy    |
| `IBrowserAdapter`      | `src/core/interfaces/browser-adapter.ts` | `name`, `launch(options) → IBrowserSession`             | Adding browser backend     |
| `IBrowserSession`      | same file                                | `newPage`, `pages`, `close`, `isConnected`              | Part of adapter            |
| `IBrowserPage`         | same file                                | Navigation, interaction, extraction, lifecycle methods  | Part of adapter            |
| `IStorage`             | `src/core/interfaces/storage.ts`         | `get`, `set`, `delete`, `list`, `clear`                 | New persistence backend    |
| `IProviderRegistry`    | `src/core/interfaces/provider.ts`        | `resolve(url)`, `get(id)`, `list`, `register`           | Custom provider resolution |

## Conventions

1. **Result pattern**: Use `ok()`, `err()`, `isOk()`, `isErr()` from `src/core/result.ts`. Never throw for expected failures.
2. **Error hierarchy**: `AuthError` subclasses in `src/core/errors.ts` — used as `Result.err()` values.
3. **ESM imports**: Always use `.js` extension (`import { X } from './foo.js'`).
4. **Strategy pattern**: Private strategy class, exported Factory with `readonly name` property.
5. **CLI command pattern**: Each command in `src/cli/commands/<name>.ts`, exported as `run<Name>(positionals, flags, deps)`. Wired in `cli/main.ts`.
6. **Adapter pattern**: Three classes (Adapter, Session, Page). Lazy-import the browser library. Throw `BrowserLaunchError` on import failure.
7. **Testing**: Vitest with `describe`/`it`/`expect`. `MemoryStorage` for isolation. Assert with `isOk()`/`isErr()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sigcli/sigcli](https://github.com/sigcli/sigcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
