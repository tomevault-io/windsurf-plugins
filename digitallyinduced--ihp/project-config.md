---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

IHP (Integrated Haskell Platform) is a batteries-included Haskell web framework built on Haskell and Nix. It follows an MVC architecture with type-safe routing, HSX templating (JSX-like syntax for HTML), and PostgreSQL as the database.

## Development Environment Setup

IHP uses devenv.sh (a wrapper around `nix develop` and `direnv`) for development:

```bash
cd /path/to/ihp
direnv allow
```

Alternatively, use `nix develop` directly to enter a dev shell. You can also use `direnv exec .` to get a cached nix develop environment.

## Running Tests

Prefer `ghci` for quick type checking and iteration — full nix builds (`nix flake check`) are slow. Use ghci interactively or via `echo ... | ghci` one-liners for fast feedback.

**When using ghci, keep in mind that it expects input on stdin. If you don't provide any, ghci will never return.** The `echo ... | ghci` pattern handles this correctly by piping input.

**IHP IDE Tests** (from the repo root):
```bash
# Interactive (allows :r to reload after changes):
ghci
:l ihp-ide/Test/Main.hs
main
:r
main

# One-liner:
echo -e ':l ihp-ide/Test/Main.hs\nmain' | ghci
```

**All tests via Nix** (slower but comprehensive):
```bash
nix flake check --impure
```

**Verify a single module compiles:**
```bash
echo ':l ihp-ide/IHP/IDE/CodeGen/MigrationGenerator.hs' | ghci
```

When adding new tests, add them to `Test/Main.hs` in the appropriate package.

## Running the Dev Server (for local IHP development)

From the IHP directory with a host project:
```bash
export DEBUG=1  # Optional: enable debug logging
ghci
:l ihp-ide/exe/IHP/IDE/DevServer.hs
mainInParentDirectory
```

## Project Structure

This is a monorepo with multiple Haskell packages:

| Package | Purpose |
|---------|---------|
| `ihp/` | Core web framework (routing, controllers, views, models, validation) |
| `ihp-ide/` | Development server, code generators, schema designer |
| `ihp-hsx/` | HSX templating system (JSX-like HTML in Haskell) |
| `ihp-migrate/` | Database migration tool |
| `ihp-datasync/` | Real-time data synchronization via WebSockets |
| `ihp-graphql/` | GraphQL API support |
| `ihp-hspec/` | Testing utilities |
| `ihp-openai/` | OpenAI integration |
| `ihp-ssc/` | Server-side components |

## Core Framework Architecture (`ihp/IHP/`)

- **Prelude modules**: `Prelude.hs`, `ControllerPrelude.hs`, `ViewPrelude.hs`, `RouterPrelude.hs`, `MailPrelude.hs` - standard imports for different contexts
- **Server.hs**: HTTP server initialization and middleware
- **FrameworkConfig.hs**: Framework configuration
- **RouterSupport.hs**: Type-safe routing and URL generation
- **QueryBuilder.hs**: Type-safe SQL query builder
- **ModelSupport.hs**: ORM-like database model support
- **Controller/**: Request handling, params, sessions, cookies, redirects
- **View/**: Form helpers, CSS framework integration, rendering
- **ValidationSupport/**: Field validation
- **AuthSupport/**: Authentication
- **LoginSupport/**: Login functionality
- **Job/**: Background job queue and runner
- **FileStorage/**: File upload abstraction
- **AutoRefresh.hs**: Real-time view updates via WebSocket
- **Mail.hs**: Email composition and sending

## IDE/Code Generation (`ihp-ide/`)

- **exe/IHP/IDE/DevServer.hs**: Development server entry point
- **IHP/IDE/SchemaDesigner/**: Visual database schema editor
- **IHP/IDE/CodeGen/**: Code generators for controllers, views, migrations, jobs, etc.
- **IHP/SchemaCompiler.hs**: Generates `Types.hs` from SQL schema

## Code Guidelines

- Use `pure` instead of `return` (clearer for non-Haskell developers)
- Add Haddock comments to public APIs
- The framework uses implicit parameters extensively (see `ImplicitParams` extension)
- HSX uses quasiquotes: `[hsx|<div>content</div>|]`

## Git Workflow

- **Never force push.** If a push is rejected because the remote has advanced, `git fetch` and rebase (or merge) onto the updated remote, then push normally. Do not use `git push --force` or `--force-with-lease`.

## Hasql Database Patterns (Reference)

Based on patterns from [hasql-tutorial1](https://github.com/nikita-volkov/hasql-tutorial1). Hasql provides type-safe PostgreSQL access with a layered architecture.

### Module Structure

The tutorial demonstrates a library-oriented structure where database code is isolated into a reusable package:

```
my-db-library/
├── my-db-library.cabal
└── library/
    └── MyDbLibrary/
        ├── Prelude.hs      -- Re-exports (rerebase), internal
        ├── Statement.hs    -- Raw SQL statements, internal
        ├── Transaction.hs  -- Business logic composition, internal
        └── Session.hs      -- PUBLIC API (only exposed module)
```

**Cabal exposure pattern:**
```cabal
library
  exposed-modules:
    MyDbLibrary.Session           -- Only public module
  other-modules:
    MyDbLibrary.Prelude           -- Internal
    MyDbLibrary.Statement         -- Internal
    MyDbLibrary.Transaction       -- Internal
```

**Why this structure?**
- **Stable API**: Only `Session` is exposed, allowing internal refactoring without breaking consumers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [digitallyinduced/ihp](https://github.com/digitallyinduced/ihp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
