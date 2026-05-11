---
trigger: always_on
description: provides:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Source of Truth

The project must always be in sync. When there are conflicts or discrepancies, the priority order for truth is:

1. **Proposals** (`Proposals/`) - The authoritative specification
2. **Code** (`Sources/`) - The implementation
3. **Documentation** ([Wiki](https://github.com/arolang/aro/wiki), `OVERVIEW.md`, `README.md`) - Developer docs
4. **Website** (`Website/`) - Public website
5. **Book** (`Book/`) - The Language Guide

When updating any layer, ensure all lower-priority layers are updated to match.

## Platform Feature Table

The **Platform Support** table in `README.md` lists feature availability for macOS, Linux, and Windows. When adding or modifying platform-specific features, always update this table to reflect current support status.

## Documentation Style

- **Proposals** (`Proposals/`): Use ASCII art for diagrams
- **Book** (`Book/`): Use SVG for diagrams

## Build Commands

```bash
swift build              # Build the project
swift test               # Run all tests
aro run ./Examples/UserService      # Run multi-file application
aro run ./Examples/HTTPServer       # Run server (uses Keepalive action)
aro compile ./MyApp   # Compile all .aro files in directory
aro check ./MyApp     # Syntax check all .aro files
aro build ./MyApp     # Compile to native binary (LLVM IR + object file)
aro build ./MyApp --verbose --optimize  # Verbose build with optimizations
```

## Architecture

This is a Swift 6.2 parser/compiler/runtime for ARO (Action Result Object), a DSL for expressing business features as Action-Result-Object statements.

### Application Structure

An ARO application is a **directory** containing `.aro` files:

```
MyApp/
├── openapi.yaml       # OpenAPI contract (required for HTTP server)
├── main.aro           # Contains Application-Start (required, exactly one)
├── users.aro          # Feature sets for user operations
├── orders.aro         # Feature sets for order operations
├── events.aro         # Event handler feature sets
├── products.store     # Seeds products-repository (read-only)
└── sessions.store     # Seeds sessions-repository (writable if chmod o+w)
```

For larger applications, use the `sources/` subdirectory convention:

```
MyApp/
├── openapi.yaml       # Configuration in root
├── main.aro           # Entry point (optional location)
└── sources/           # Source files in subdirectory
    ├── users/
    │   └── users.aro
    └── orders/
        └── orders.aro
```

**Key Rules:**
- All `.aro` files in the directory **and subdirectories** are automatically discovered and parsed
- Files can be in root, `sources/`, or any subdirectory to any depth
- No imports needed - all feature sets are globally visible within the application
- Exactly ONE `Application-Start` feature set per application (error if 0 or multiple)
- At most ONE `Application-End: Success` and ONE `Application-End: Error` (both optional)
- Feature sets are triggered by **events**, not direct calls
- **Contract-First HTTP**: `openapi.yaml` is required for HTTP server (no contract = no server)

### Compilation Pipeline

```
Directory → Find all .aro files → Compile each → Validate single Application-Start → Register with EventBus
```

- **Lexer** (`Lexer.swift`): Tokenizes source, recognizing articles (a/an/the), prepositions, and compound identifiers
- **Parser** (`Parser.swift`): Recursive descent parser producing AST
- **SemanticAnalyzer** (`SemanticAnalyzer.swift`): Builds symbol tables and performs data flow analysis
- **Compiler** (`Compiler.swift`): Orchestrates the pipeline, entry point is `Compiler.compile(source)`

### Runtime Execution

```
Application-Start executes → Services start → Event loop waits → Events trigger feature sets
```

- **ApplicationLoader**: Discovers and compiles all `.aro` files in directory
- **ExecutionEngine** (`Core/ExecutionEngine.swift`): Orchestrates program execution
- **EventBus** (`Events/EventBus.swift`): Routes events to matching feature sets
- **FeatureSetExecutor** (`Core/FeatureSetExecutor.swift`): Executes feature sets when triggered
- **ActionRegistry** (`Actions/ActionRegistry.swift`): Maps verbs to implementations

### Event-Driven Feature Sets

Feature sets are triggered by events based on their **business activity**:

| Business Activity Pattern | Triggered By |
|---------------------------|--------------|
| `operationId` (e.g., `listUsers`) | HTTP route match via OpenAPI contract |
| `{EventName} Handler` | Custom domain events |
| `{repository-name} Observer` | Repository changes (store/update/delete) |
| `File Event Handler` | File system events |
| `Socket Event Handler` | Socket events |

### Contract-First HTTP APIs

ARO uses **contract-first** API development. HTTP routes are defined in `openapi.yaml`, and feature sets are named after `operationId` values.

**Without openapi.yaml**: HTTP server does NOT start, no port is opened.
**With openapi.yaml**: HTTP server is enabled and routes are handled.

Example:
```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      operationId: listUsers    # Feature set name
    post:
      operationId: createUser
  /users/{id}:
    get:
      operationId: getUser
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arolang/aro](https://github.com/arolang/aro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
