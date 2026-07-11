---
trigger: always_on
description: SpacetimeDB reference: reducers, tables, subscriptions, scheduling, and RLS patterns. Apply when implementing or reviewing server module logic, schema changes, generated bindings impacts, or client callback/dataflow behavior tied to SpacetimeDB.
---


# SpacetimeDB

> SpacetimeDB is a fully-featured relational database system that integrates
> application logic directly within the database, eliminating the need for
> separate web or game servers. It supports multiple programming languages,
> including C# and Rust, allowing developers to write and deploy entire
> applications as a single binary. It is optimized for high-throughput and low
> latency multiplayer applications like multiplayer games.

Users upload their application logic to run inside SpacetimeDB as a WebAssembly
module. There are three main features of SpacetimeDB: tables, reducers, and
subscription queries. Tables are relational database tables like you would find
in a database like Postgres. Reducers are atomic, transactional, RPC functions
that are defined in the WebAssembly module which can be called by clients.
Subscription queries are SQL queries which are made over a WebSocket connection
which are initially evaluated by SpacetimeDB and then incrementally evaluated
sending changes to the query result over the WebSocket.

All data in the tables are stored in memory, but are persisted to the disk via a
Write-Ahead Log (WAL) called the Commitlog. All tables are persistent in
SpacetimeDB.

SpacetimeDB allows users to code generate type-safe client libraries based on
the tables, types, and reducers defined in their module. Subscription queries
allows the client SDK to store a partial, live updating, replica of the servers
state. This makes reading database state on the client extremely low-latency.

Authentication is implemented in SpacetimeDB using the OpenID Connect protocol.
An OpenID Connect token with a valid `iss`/`sub` pair constitutes a unique and
authenticable SpacetimeDB identity. SpacetimeDB uses the `Identity` type as an
identifier for all such identities. `Identity` is computed from the `iss`/`sub`
pair using the following algorithm:

1. Concatenate the issuer and subject with a pipe symbol (`|`).
2. Perform the first BLAKE3 hash on the concatenated string.
3. Get the first 26 bytes of the hash (let's call this `idHash`).
4. Create a 28-byte sequence by concatenating the bytes `0xc2`, `0x00`, and `idHash`.
5. Compute the BLAKE3 hash of the 28-byte sequence from step 4 (let's call this `checksumHash`).
6. Construct the final 32-byte `Identity` by concatenating: the two prefix bytes (`0xc2`, `0x00`), the first 4 bytes of `checksumHash`, and the 26-byte `idHash`.
7. This final 32-byte value is typically represented as a hexadecimal string.

```ascii
Byte Index: |  0  |  1  |  2  |  3  |  4  |  5  |  6  |  7  | ... | 31  |
            +-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
Contents:   | 0xc2| 0x00| Checksum Hash (4 bytes) |  ID Hash (26 bytes) |
            +-----+-----+-------------------------+---------------------+
                      (First 4 bytes of           (First 26 bytes of
                       BLAKE3(0xc200 || idHash))    BLAKE3(iss|sub))
```

This allows SpacetimeDB to easily integrate with OIDC authentication
providers like FirebaseAuth, Auth0, or SuperTokens.

Clockwork Labs, the developers of SpacetimeDB, offers three products:

1. SpacetimeDB Standalone: a source available (Business Source License), single node, self-hosted version
2. SpacetimeDB Maincloud: a hosted, managed-service, serverless cluster
3. SpacetimeDB Enterprise: a closed-source, clusterized version of SpacetimeDB which can be licensed for on-prem hosting or dedicated hosting

## Documentation Directory

### Getting Started
- [What is SpacetimeDB](/intro/what-is-spacetimedb) - Overview and core concepts
- [Key Architecture](/intro/key-architecture) - How SpacetimeDB works
- [Language Support](/intro/language-support) - Supported languages and SDKs
- [FAQ](/intro/faq) - Frequently asked questions

### Quickstarts
- [React Quickstart](/quickstarts/react) - Get started with React + TypeScript
- [TypeScript Quickstart](/quickstarts/typescript) - TypeScript server module
- [Rust Quickstart](/quickstarts/rust) - Rust server module
- [C# Quickstart](/quickstarts/c-sharp) - C# server module

### Core Concepts
- [Databases](/databases) - Database modules overview
- [Tables](/tables) - Defining and working with tables
  - [Columns](/tables/columns) - Column types and definitions
  - [Indexes](/tables/indexes) - Creating and using indexes
  - [Scheduled Tables](/tables/scheduled-tables) - Time-based scheduling
  - [Access Permissions](/tables/access-permissions) - Table visibility (public/private)
- [Functions](/functions) - Server-side logic
  - [Reducers](/functions/reducers) - Transactional RPC functions
  - [Reducer Context](/functions/reducers/reducer-context) - ctx.db, ctx.sender, etc.
  - [Lifecycle Reducers](/functions/reducers/lifecycle) - init, client_connected, client_disconnected
  - [Error Handling](/functions/reducers/error-handling) - Handling errors in reducers
  - [Procedures](/functions/procedures) - Non-transactional functions with side effects
  - [Views](/functions/views) - Computed data views
- [Subscriptions](/subscriptions) - Real-time data synchronization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeloSlav/medieval-city-builder-starter-kit-threejs](https://github.com/SeloSlav/medieval-city-builder-starter-kit-threejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
