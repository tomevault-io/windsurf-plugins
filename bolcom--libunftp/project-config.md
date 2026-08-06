---
trigger: always_on
description: This repository is a Rust Cargo workspace for libunftp, an asynchronous and extensible FTP(S) server library.
---

# AGENTS.md

## Repository overview

This repository is a Rust Cargo workspace for libunftp, an asynchronous and extensible FTP(S) server library.

Workspace members:

- `libunftp` at the repository root: FTP server implementation and public server builder.
- `crates/unftp-core`: public service-provider interfaces shared by the server and backend crates.
- `crates/unftp-auth-jsonfile`: JSON-file authentication backend.
- `crates/unftp-auth-pam`: PAM authentication backend.
- `crates/unftp-auth-rest`: REST authentication backend.
- `crates/unftp-sbe-fs`: local filesystem storage backend.
- `crates/unftp-sbe-gcs`: Google Cloud Storage backend.

Authentication and storage extension interfaces belong in `unftp-core`. The root `libunftp` crate consumes those interfaces and implements FTP server behavior.

## Architecture

### Public API and server construction

`ServerBuilder<Storage, User>` in `src/server/ftpserver.rs` is the main public configuration API. `ServerBuilder::build()` validates configuration and creates a `Server`.

Public backend traits and related types are defined in `unftp-core`:

- Authentication APIs: `crates/unftp-core/src/auth`
- Storage APIs: `crates/unftp-core/src/storage`

Avoid moving backend-facing APIs into internal server modules. Backend implementations should depend on `unftp-core`, not on libunftp internals.

### Control channel

Control-channel code lives under `src/server/controlchan`.

Incoming FTP lines follow this path:

1. `FtpCodec` decodes a line.
2. `line_parser` parses it into a `Command`.
3. The command becomes an `Event`.
4. A middleware chain applies metrics, logging, FTPS policy, authentication policy, active/passive-mode policy, and notifications.
5. `PrimaryEventHandler` selects the command-specific `CommandHandler`.
6. The handler returns an FTP `Reply` or sends an internal message for asynchronous completion.

FTP commands normally have:

- A `Command` variant in `controlchan/command.rs`
- Parsing logic in `controlchan/line_parser/parser.rs`
- Parser tests in `controlchan/line_parser/tests.rs`
- A handler module in `controlchan/commands`
- Registration and re-export in `controlchan/commands/mod.rs`
- Dispatch logic in `controlchan/control_loop.rs`

Commands involving data transfer also interact with the data-channel command types and executor.

### Session and concurrency model

Per-connection state is stored in `Session<Storage, User>` and shared as an `Arc<tokio::sync::Mutex<_>>`.

Control- and data-channel tasks communicate through Tokio channels using the message types in `src/server/chancomms.rs`. Storage instances and authenticated user details are shared through `Arc`.

Do not hold a session mutex guard across slow storage or network operations unless the existing design specifically requires it. Copy or clone the required session values first and release the guard.

### Data channel

Data-transfer behavior lives primarily in `src/server/datachan.rs`.

The data channel handles commands such as:

- `RETR`
- `STOR`
- `APPE`
- `LIST`
- `NLST`
- `MLSD`

Transfer completion and storage failures are reported to the control channel through `ControlChanMsg`.

### Listener modes

Listener implementations live under `src/server/ftpserver`.

Supported modes include:

- Legacy listener mode
- Pooled listener mode
- Proxy Protocol mode when the `proxy_protocol` feature is enabled

Pooled and proxy modes use the switchboard in `src/server/switchboard.rs` to associate passive data connections with control sessions.

### Authentication

Authentication is split into two stages:

1. An `Authenticator` converts credentials into a `Principal`.
2. A `UserDetailProvider` converts the principal into the application's `UserDetail` type.

The root crate contains the server-side authentication pipeline. Reusable authentication interfaces belong in `unftp-core`.

### Storage capabilities

Storage backends implement `StorageBackend<User>` and its associated `Metadata` type.

Optional backend capabilities are advertised by `StorageBackend::supported_features()` using the `FEATURE_*` constants in `unftp-core`. The server uses these flags when advertising or accepting optional FTP behavior.

## Coding conventions

### Rust and formatting

- Workspace crates use Rust edition 2024.
- Follow `rustfmt.toml`.
- Use four-space indentation.
- The configured maximum line width is 160.
- Let rustfmt reorder imports.
- Prefer field-init shorthand and `?` shorthand where applicable.
- Do not introduce unsafe code. The workspace denies `unsafe_code`.

Run rustfmt rather than manually approximating its output.

### Lints and documentation

The workspace denies:

- Missing documentation on public APIs
- All Clippy lints configured by the `all` lint group

Every new public type, trait, method, field, enum variant, and relevant module must have useful rustdoc.

Public examples should compile as doctests unless they are explicitly marked otherwise. Update examples and documentation when changing a public API.

### Async code

- The runtime is Tokio.
- Async extension traits commonly use `async_trait`.
- Trait objects and values crossing spawned tasks generally require `Send`, `Sync`, and `'static` bounds.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bolcom/libunftp](https://github.com/bolcom/libunftp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
