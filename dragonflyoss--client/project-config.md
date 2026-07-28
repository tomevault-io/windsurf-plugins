---
trigger: always_on
description: Dragonfly Client (`dragonflyoss/client`) is a high-performance P2P download client written in **Rust 1.88.0**. It implements both the peer and seed-peer roles in the [Dragonfly](https://d7y.io/) distributed file distribution system. The daemon (`dfdaemon`) intercepts HTTP requests via an MITM proxy, coordinates piece-level downloads across peers, and persists data through a local RocksDB-backed storage layer.
---

# GitHub Copilot Instructions for Dragonfly Client

## Project Overview

Dragonfly Client (`dragonflyoss/client`) is a high-performance P2P download client written in **Rust 1.88.0**. It implements both the peer and seed-peer roles in the [Dragonfly](https://d7y.io/) distributed file distribution system. The daemon (`dfdaemon`) intercepts HTTP requests via an MITM proxy, coordinates piece-level downloads across peers, and persists data through a local RocksDB-backed storage layer.

---

## Workspace Structure

The repository is a Cargo workspace (resolver `"2"`) containing nine crates:

| Crate                                      | Purpose                                                                    |
| ------------------------------------------ | -------------------------------------------------------------------------- |
| `dragonfly-client`                         | Main binaries: `dfdaemon`, `dfget`, `dfcache`, `dfstore`, `dfctl`          |
| `dragonfly-client-core`                    | Shared `DFError` enum, `Result<T>` alias, `OrErr` trait                    |
| `dragonfly-client-config`                  | Configuration structs and validation for all binaries                      |
| `dragonfly-client-storage`                 | RocksDB metadata, content files, LRU cache, TCP/QUIC servers               |
| `dragonfly-client-backend`                 | Download backends: HTTP, S3/object-storage, HDFS, HuggingFace, plugins     |
| `dragonfly-client-util`                    | Utilities: crypto, TLS, networking, ID generation, rate limiting, shutdown |
| `dragonfly-client-metric`                  | Prometheus metrics                                                         |
| `dragonfly-client-request`                 | Request library for downloading files via Dragonfly HTTP Proxy             |
| `dragonfly-client-init`                    | `dfinit` binary — bootstraps the runtime environment                       |
| `dragonfly-client-backend/examples/plugin` | Example backend plugin                                                     |

All crates share `[workspace.package]` metadata (version `1.2.11`, Apache-2.0 license, edition `2021`) and `[workspace.dependencies]` for consistent dependency versions.

---

## Rust Conventions

### Toolchain

Pin the toolchain to `1.88.0` via `rust-toolchain.toml`. Always write code compatible with that version.

### Edition

Use Rust **2021 edition** features throughout.

### Formatting

Format all code with `cargo fmt --all`. There is no custom `rustfmt.toml`; use the default Rustfmt settings.

### Linting

All warnings are errors: `cargo clippy --all --all-targets -- -D warnings`. Fix every Clippy warning before submitting.

### File License Header

Every `.rs` source file must start with the Apache 2.0 license header:

```rust
/*
 *     Copyright 2024 The Dragonfly Authors
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```

### Naming

Follow standard Rust naming conventions: `snake_case` for functions, variables, and modules; `PascalCase` for types and traits; `SCREAMING_SNAKE_CASE` for constants.

---

## Error Handling

### Core Error Type

All errors flow through `DFError` defined in `dragonfly-client-core/src/error/mod.rs`:

```rust
use dragonfly_client_core::{Error, Result as ClientResult};
```

`Result<T>` is a type alias for `std::result::Result<T, DFError>`. Use `ClientResult<T>` (or `Result<T>` after the local alias) everywhere — **never use `anyhow` in library code**.

### `OrErr` Trait

Use the `OrErr` trait to attach an `ErrorType` when converting external errors:

```rust
use dragonfly_client_core::error::{ErrorType, OrErr};

let value = some_operation().or_err(ErrorType::SerializeError)?;
```

### Error Propagation

Propagate errors with `?`. Log at the site of handling, not at every propagation step.

```rust
let result = operation()
    .inspect_err(|err| error!("operation failed: {}", err))?;
```

### Avoiding Panics

Do **not** use `unwrap()` or `expect()` in production code paths. Reserve them for tests and truly unreachable branches, where you should add a comment explaining why the invariant holds.

---

## Logging and Tracing

### Framework

Use the `tracing` crate with structured fields. Import macros directly:

```rust
use tracing::{debug, error, info, instrument, warn, Instrument, Span};
```

### Log Levels


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dragonflyoss/client](https://github.com/dragonflyoss/client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
