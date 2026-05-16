---
trigger: always_on
description: > SpacetimeDB is a fully-featured relational database system that integrates
---

# SpacetimeDB

> SpacetimeDB is a fully-featured relational database system that integrates
application logic directly within the database, eliminating the need for
separate web or game servers. It supports multiple programming languages,
including C# and Rust, allowing developers to write and deploy entire
applications as a single binary. It is optimized for high-throughput and low
latency multiplayer applications like multiplayer games.

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

## Basic Project Workflow

Getting started with SpacetimeDB involves a few key steps:

1.  **Install SpacetimeDB:** Install the `spacetime` CLI tool for your operating system. This tool is used for managing modules, databases, and local instances.

    *   **macOS:**
        ```bash
        curl -sSf https://install.spacetimedb.com | sh
        ```
    *   **Windows (PowerShell):**
        ```powershell
        iwr https://windows.spacetimedb.com -useb | iex
        ```
    *   **Linux:**
        ```bash
        curl -sSf https://install.spacetimedb.com | sh
        ```
    *   **Docker (to run the server):**
        ```bash
        # This command starts a SpacetimeDB server instance in Docker
        docker run --rm --pull always -p 3000:3000 clockworklabs/spacetime start 
        # Note: While the CLI can be installed separately (see above), you can also execute 
        # CLI commands *within* the running Docker container (e.g., using `docker exec`) 
        # or use the image as a base for a custom image containing your module management tools.
        ```
    *   **Docker (to execute CLI commands directly):**
        You can also use the Docker image to run `spacetime` CLI commands without installing the CLI locally. For commands that operate on local files (like `build`, `publish`, `generate`), this involves mounting your project directory into the container. For commands that only interact with a database instance (like `sql`, `status`), mounting is typically not required, but network access to the database is.
        ```bash
        # Example: Build a module located in the current directory (.)
        # Mount current dir to /module inside container, set working dir to /module
        docker run --rm -v "$(pwd):/module" -w /module clockworklabs/spacetime build --project-path .


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thinktidevibes/2D-Survival-Multiplayer-Game](https://github.com/thinktidevibes/2D-Survival-Multiplayer-Game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
