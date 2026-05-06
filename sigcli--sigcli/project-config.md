---
trigger: always_on
description: Monorepo for SigCLI authentication tools. pnpm workspaces, ESM, Node >= 18.
---

# Sigcli

Monorepo for SigCLI authentication tools. pnpm workspaces, ESM, Node >= 18.

When you make this project, do not add you as an author in commit. e.g. DO NOT add this in commit message "Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"

## Monorepo Structure

```
sigcli/
├── cli/           # @sigcli/cli — CLI with pluggable strategies and browser adapters
├── sdk/
│   ├── typescript/  # @sigcli/sdk — lightweight client SDK (npm)
│   └── python/      # sigcli-sdk — lightweight client SDK (PyPI)
├── website/       # sigcli.ai — TanStack Start + React 19
├── skills/        # AI agent skills using SigCLI (any language)
├── docs/          # Documentation
└── pitch/         # Pitch materials
```

## Commands

```bash
pnpm install               # Install all workspace deps
pnpm -r build              # Build all packages
pnpm -r test               # Test all packages
pnpm --filter @sigcli/cli build   # Build CLI only
pnpm --filter @sigcli/cli test    # Test CLI only
pnpm --filter @sigcli/sdk build  # Build TS SDK only
pnpm --filter website dev          # Dev server for website
```

## CLI Package (`cli/`)

### Architecture

```
bin/sig.js (entry)  ──▶  cli/main.ts (router)  ──▶  cli/commands/*
                                                             │
deps.ts (composition root) ── wires all deps via DI ────────▶ AuthManager ──▶ Strategies + Storage + Browser
                                                             │
core/ (types, interfaces, Result, errors) ── zero external deps, imported by all layers
```

- **`bin/sig.js`** — Entry point. Auto-builds if needed, delegates to CLI router.
- **`src/deps.ts`** — Composition root. Creates registries, storage, browser factory, AuthManager. No singletons.
- **`src/auth-manager.ts`** — Orchestrator. Flow: stored cred → validate → refresh → authenticate. All methods return `Result<T, AuthError>`.
- **`src/core/`** — Shared vocabulary. Zero external dependencies.
- **`src/cli/`** — CLI commands (init, doctor, get, login, request, status, logout, providers, remote, sync, watch, rename, remove, completion).
- **`src/strategies/`** — Each strategy: private class + exported `*StrategyFactory` (IAuthStrategyFactory).
- **`src/browser/adapters/`** — Browser automation. Three-class pattern: Adapter → Session → Page.
- **`src/browser/flows/`** — `runHybridFlow`, `extractOAuthTokens`, `isLoginPage`, `startHeaderCapture`.
- **`src/storage/`** — DirectoryStorage (per-file JSON + file lock + AES-256-GCM encryption), CachedStorage, MemoryStorage.
- **`src/crypto/`** — Encryption at rest. AES-256-GCM encrypt/decrypt, key generation/loading. Key stored at `~/.sig/encryption.key`.
- **`src/providers/`** — ProviderRegistry, config-loader.
- **`src/sync/`** — SyncEngine + SshTransport (encrypts with remote key).
- **`src/proxy/`** — MITM proxy daemon. CaManager (ECDSA P-256 CA + per-hostname leaf certs), ProxyServer (HTTP/HTTPS CONNECT with credential injection), daemon (proxy + watch loop), proxy-state (PID/port files at `~/.sig/proxy/`).
- **`src/utils/`** — JWT decode, duration parse, HTTP helpers.

### Key Interfaces

| Interface              | Location                                 | Methods                                                 |
| ---------------------- | ---------------------------------------- | ------------------------------------------------------- |
| `IAuthStrategy`        | `src/core/interfaces/auth-strategy.ts`   | `validate`, `authenticate`, `refresh`, `applyToRequest` |
| `IAuthStrategyFactory` | same file                                | `name`, `create(config)`                                |
| `IBrowserAdapter`      | `src/core/interfaces/browser-adapter.ts` | `name`, `launch(options) → IBrowserSession`             |
| `IBrowserSession`      | same file                                | `newPage`, `pages`, `close`, `isConnected`              |
| `IBrowserPage`         | same file                                | Navigation, interaction, extraction, lifecycle methods  |
| `IStorage`             | `src/core/interfaces/storage.ts`         | `get`, `set`, `delete`, `list`, `clear`                 |
| `IProviderRegistry`    | `src/core/interfaces/provider.ts`        | `resolve(url)`, `get(id)`, `list`, `register`           |

### Conventions

1. **Result pattern**: Use `ok()`, `err()`, `isOk()`, `isErr()` from `src/core/result.ts`. Never throw for expected failures.
2. **Error hierarchy**: `AuthError` subclasses in `src/core/errors.ts`.
3. **ESM imports**: Always use `.js` extension.
4. **Strategy pattern**: Private strategy class, exported Factory with `readonly name` property.
5. **CLI command pattern**: Each command in `src/cli/commands/<name>.ts`, exported as `run<Name>(positionals, flags, deps)`.
6. **Adapter pattern**: Three classes (Adapter, Session, Page). Lazy-import the browser library.
7. **Testing**: Vitest with `describe`/`it`/`expect`. `MemoryStorage` for isolation.
8. **Credential types**: Discriminated union on `type` field: `'cookie' | 'bearer' | 'api-key' | 'basic'`.
9. **Config**: `StrategyConfig = Record<string, unknown>` — parsed inside each strategy via a private `parseConfig()`.
10. **Exports**: Public API through `src/index.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sigcli/sigcli](https://github.com/sigcli/sigcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
