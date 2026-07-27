---
trigger: always_on
description: Guide for AI agents (Claude, Codex, Cline, …) working on the macOS Swift codebase. Scope: everything under [macos/](.). Windows and Linux ports have their own conventions.
---

# AGENTS.md — Gridex macOS

Guide for AI agents (Claude, Codex, Cline, …) working on the macOS Swift codebase. Scope: everything under [macos/](.). Windows and Linux ports have their own conventions.

**Read order on every new task:**
1. This file.
2. [`CLAUDE.md`](../CLAUDE.md) at the repo root (project overview + behavioural guard-rails — `Think before coding`, `Surgical changes`, `Goal-driven execution`).
3. The closest existing adapter/service to what you're touching — copy its shape before inventing your own.

If anything in this file conflicts with `CLAUDE.md`, the root `CLAUDE.md` wins. This file is the **macOS-specific** layer on top.

---

## 0. Architecture invariants — non-negotiable

These are load-bearing. Don't violate them without an explicit user instruction.

- **5-layer Clean Architecture, dependencies point inward only.**
  `Core ← Domain ← Data ← Services ← Presentation`. Core has zero dependencies. Data implements Domain. Presentation never imports Data directly.
- **Single `DependencyContainer.shared`** at [`App/DependencyInjection/DependencyContainer.swift`](App/DependencyInjection/DependencyContainer.swift) is the only composition root. Don't `init` services elsewhere; ask the container.
- **Thread-safe services are `actor`.** `ConnectionManager`, `QueryEngine`, `MCPServer`, `SSHTunnelService`, `SchemaInspectorService`, `ProviderRegistry`, `BackupService` are all actors. Cross-actor types must be `Sendable`.
- **Domain models are plain Swift** (`struct`, `Codable`, `Sendable`). SwiftData `@Model` entities live only under [`Data/Persistence/Models/`](Data/Persistence/Models/) and round-trip via `toConfig()` / equivalent.
- **Errors are `GridexError`.** Never throw raw `NSError` or driver errors out of an adapter. Wrap with the matching case.
- **Credentials live in Keychain.** `KeychainService` (via `keychainService` on the container). Passwords are passed into adapters in-memory only; they are never on `ConnectionConfig`.
- **No global mutable state besides `DependencyContainer.shared` and `AppState`.** Per-window state goes on `AppState`; global on the container.

---

## 1. Reusable surface — use these before writing new code

Before introducing a new helper/protocol/model, scan this section. Hidden coupling notes in `[Coupling]` callouts.

### 1.1 Core protocols — the contracts

| Protocol | File | Conform when… |
|---|---|---|
| `DatabaseAdapter` | [Core/Protocols/Database/DatabaseAdapter.swift](Core/Protocols/Database/DatabaseAdapter.swift) | Adding a new database engine. ~50 methods covering connect/disconnect, execute (incl. parameterized), schema introspection, CRUD, pagination, transactions. |
| `SchemaInspectable` | [Core/Protocols/Database/SchemaInspectable.swift](Core/Protocols/Database/SchemaInspectable.swift) | The new adapter can introspect schemas. Powers AI context + ER diagram. |
| `LLMService` | [Core/Protocols/AI/LLMService.swift](Core/Protocols/AI/LLMService.swift) | Adding a new AI provider. Streaming-first, `AsyncThrowingStream`. |
| `AIContextProvider` | [Core/Protocols/AI/AIContextProvider.swift](Core/Protocols/AI/AIContextProvider.swift) | You almost certainly don't need a new one — reuse `aiContextEngine` from the container. |
| `MCPTool` | `Services/MCP/Tools/` | Adding a new MCP tool. Must declare permission tier + input schema. |

### 1.2 Core models — the value types every layer exchanges

| Model | File | Notes |
|---|---|---|
| `RowValue` | [Core/Models/Database/RowValue.swift](Core/Models/Database/RowValue.swift) | Unified cell type (null/string/int/double/bool/date/uuid/json/data/array). Always use; never leak driver-native types upward. |
| `ConnectionConfig` | [Core/Models/Database/ConnectionConfig.swift](Core/Models/Database/ConnectionConfig.swift) | Canonical connection spec. Includes `mongoOptions: [String:String]?` (URI query params), SSH tunnel config, mTLS cert paths. **Password is NOT a field** — passed separately. |
| `QueryResult` | [Core/Models/Query/QueryResult.swift](Core/Models/Query/QueryResult.swift) | Standard query output. `columns + rows + rowsAffected + executionTime + queryType`. |
| `QueryParameter` | [Core/Models/Query/QueryParameter.swift](Core/Models/Query/QueryParameter.swift) | Wraps `RowValue` for parameterized execution. Adapters bind via dialect-specific placeholders. |
| `FilterExpression` | [Core/Models/Query/FilterExpression.swift](Core/Models/Query/FilterExpression.swift) | Grid filter UI → SQL. `toSQL(dialect:)` builds the WHERE clause. **Use this; never concat WHERE strings by hand.** |
| `QuerySortDescriptor` | [Core/Models/Query/SortDescriptor.swift](Core/Models/Query/SortDescriptor.swift) | Grid sort state → ORDER BY. |
| `SchemaSnapshot`, `TableDescription`, `ColumnInfo`, `IndexInfo`, `ForeignKeyInfo`, `ViewInfo`, `TableInfo` | [Core/Models/Schema/](Core/Models/Schema/) | Schema introspection output. `TableDescription.toDDL(dialect:)` regenerates CREATE TABLE for export. |
| `AIContext`, `LLMMessage`, `ChatMessage` | [Core/Models/AI/AIModels.swift](Core/Models/AI/AIModels.swift) | AI chat data model (token-budgeted context, role messages, persisted history with inlined SQL/results). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gridex/gridex](https://github.com/gridex/gridex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
