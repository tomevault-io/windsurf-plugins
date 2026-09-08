---
trigger: always_on
description: Instructions for coding agents working in this repo.
---

# AGENTS.md

Instructions for coding agents working in this repo.

Companion documents:

- [CONTEXT.md](CONTEXT.md) - glossary of the domain language (Statement, Params, OID cache, statement cache, …). Read it before naming anything; naming that contradicts the glossary is a defect.
- <https://github.com/nikita-volkov/haskell-coding-standards> - the general Haskell design system this project follows (imports, exports, naming, errors, deriving, formatting, documentation, architecture patterns). This file records only what is *specific to hasql*; for anything general, defer to the standards repo.

## Project Overview

Hasql is a fast PostgreSQL driver with a flexible mapping API. It is the root of a granular ecosystem of composable libraries, each staying simple and doing one thing. The project favours modularity, type safety, and explicit error handling over exceptions.

### Ecosystem Approach

- **Modular design** - an ecosystem of small focused libraries rather than one monolith.
- **Horizontal scalability** - users are encouraged to write extension libraries rather than grow the core.
- **Composability** - each library exposes a simple API that combines with the others.
- **Interchangeability** - several libraries may solve the same problem in different ways.

### Key Abstractions

- **Connection** - manages a database connection, its settings, OID cache and statement cache.
- **Session** - a batch of actions executed in a connection context. `Hasql.Engine.Contexts.Session` derives its instances `via (ExceptT SessionError (StateT ConnectionState IO))`.
- **Pipeline** - composable abstraction for executing several queries in one round trip.
- **Statement** - a single SQL query plus its parameter encoder and result decoder.
- **Encoders** - DSL for declaring parameter encoders (Params, Value, NullableOrNot).
- **Decoders** - DSL for declaring result decoders (Result, Row, Value, NullableOrNot).

### Layers

The codebase is layered along two axes: cabal components, and namespaces inside the `library` component. Allowed dependency edges:

| Layer | May depend on |
|---|---|
| `platform` | — |
| `to-be-resolved` | — |
| `codecs-vocab` | `platform`, `to-be-resolved` |
| `comms` | `platform` |
| `connection-state` | `codecs-vocab`, `platform` |
| `Hasql.Codecs.*` | `codecs-vocab`, `platform`, `to-be-resolved` |
| `Hasql.Engine.*` | `Hasql.Codecs.*`, `codecs-vocab`, `comms`, `connection-state`, `platform`, `to-be-resolved` |

`codecs-vocab`, `comms`, `connection-state`, `to-be-resolved` and `platform` are separate cabal components; `Hasql.Codecs.*` and `Hasql.Engine.*` are namespaces inside the `library` component. Namespaces not listed (`Hasql.Connection.*`, the top-level public modules) are deliberately left unconstrained.

Cabal components:

```mermaid
flowchart BT
  platform
  to-be-resolved
  codecs-vocab --> platform
  codecs-vocab --> to-be-resolved
  comms --> platform
  connection-state --> codecs-vocab
  connection-state --> platform
  library --> codecs-vocab
  library --> comms
  library --> connection-state
  library --> platform
  library --> to-be-resolved
```

`Hasql.*` namespaces within the `library` component:

```mermaid
flowchart BT
  Codecs --> codecs-vocab
  Codecs --> platform
  Codecs --> to-be-resolved
  Engine --> Codecs
  Engine --> codecs-vocab
  Engine --> comms
  Engine --> connection-state
  Engine --> platform
  Engine --> to-be-resolved
```

- `Platform/` - the custom prelude and shared primitives.
- `Codecs/` - encoder and decoder DSLs.
- `Comms/` - protocol round trips and result decoding, on top of the libpq binding.
- `Engine/` - statement compilation, result/row decoding, and contexts (Session, Pipeline) that drive `connection-state`.
- Top-level modules (`Hasql.Connection`, `Hasql.Session`, …) are the public API.

Postgres itself is reached through the external [pqi](https://github.com/nikita-volkov/pqi) library, which abstracts over interchangeable adapters (`pqi-ffi`, `pqi-native`). Hasql does not carry its own libpq bindings - do not reintroduce any.

## Code Style & Conventions

### Language Extensions

Defined once in the `common base` stanza of [hasql.cabal](hasql.cabal) and imported by every component. Consult that stanza rather than assuming; do not add a per-module `{-# LANGUAGE #-}` pragma for an extension that belongs in the shared list.

### Imports

- **Qualified imports** for everything except the module's own topic - e.g. `qualified as Encoders`, `qualified as Decoders`.
- Qualify by the module's topic, not by an arbitrary abbreviation. Self-qualified form (`import Pqi.Ffi qualified`) is preferred where the name is already short.
- **Custom prelude** - every library module imports `Hasql.Platform.Prelude`, never the standard `Prelude`. Outside the library the convention follows the layer under test: `library-tests`, `connection-state-tests`, `benchmarks` and `profiling` use plain `Prelude`.

### Naming

- **Newtype wrappers** are used extensively for type safety (Session, Statement, Connection). Prefer a newtype over a raw primitive whenever the value carries a domain meaning - a bare `Word32` that is really an OID, or an `Int32` that is really a row index, is a defect.
- **DSL style** for encoders and decoders.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikita-volkov/hasql](https://github.com/nikita-volkov/hasql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
