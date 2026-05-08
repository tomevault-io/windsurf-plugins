---
trigger: always_on
description: Self-contained in-memory service providing "working memory" for AI agents. Combines a fast key-value cache with semantic vector search in a single process, accessible via Redis-compatible RESP protocol.
---

# Agentis Memory — CLAUDE.md

## Project Overview

Self-contained in-memory service providing "working memory" for AI agents. Combines a fast key-value cache with semantic vector search in a single process, accessible via Redis-compatible RESP protocol.

**Core principle:** One binary, one process, one port. Zero dependencies. Download and run.

**Key differentiator:** Redis-compatible wire protocol for agent memory. Any Redis client (redis-cli, Jedis, Lettuce, redis-py) works out of the box.

## Tech Stack

- **Language:** Java 26
- **Build:** GraalVM native-image → single binary (~100-150MB)
- **Embedding:** ONNX Runtime via Panama FFI, all-MiniLM-L6-v2 (~80MB, 384 dim), bundled in binary
- **Vector index:** jvector (DataStax, Apache 2.0) — HNSW index, cosine similarity
- **SIMD:** Java Vector API for cosine similarity acceleration
- **Network:** Netty with io_uring (Linux) / kqueue (macOS)
- **Protocol:** RESP v2 (Redis wire protocol)

## Repository Structure

```
agentis-memory/
├── CLAUDE.md
├── build.gradle.kts
├── gradle.properties                                  # dependency versions
├── agentis-memory.conf.example
├── docs/
│   └── superpowers/
│       └── specs/
│           └── 2026-03-26-agentis-memory-design.md   # Full design spec
├── models/                                            # Bundled ONNX model artifacts
│   ├── model.onnx                                     # all-MiniLM-L6-v2 (~80MB)
│   ├── tokenizer.json
│   └── tokenizer_config.json
└── src/
    ├── main/java/io/agentis/memory/
    │   ├── AgentisMemory.java              # main(), startup, shutdown hook
    │   ├── config/
    │   │   └── ServerConfig.java           # CLI args + conf file parsing
    │   ├── resp/
    │   │   ├── RespMessage.java            # sealed interface: SimpleString, Error, Integer, BulkString, Array
    │   │   ├── RespDecoder.java            # Netty ByteToMessageDecoder
    │   │   ├── RespEncoder.java            # Netty MessageToByteEncoder
    │   │   ├── RespServer.java             # Netty bootstrap, pipeline setup
    │   │   └── CommandDispatcher.java      # Netty handler → CommandRouter
    │   ├── command/
    │   │   ├── CommandHandler.java         # interface: handle(ctx, args) → RespMessage
    │   │   ├── CommandRouter.java          # command name → handler dispatch
    │   │   ├── kv/                         # standard Redis commands
    │   │   │   ├── SetCommand.java
    │   │   │   ├── GetCommand.java
    │   │   │   └── PingCommand.java
    │   │   └── mem/                        # (Future Layer 2)
    │   ├── store/
    │   │   ├── Entry.java                  # record: value, createdAt, expireAt, hasVectorIndex
    │   │   └── KvStore.java                # ConcurrentHashMap<String, Entry>
    │   └── vector/
    │       └── Embedder.java               # ONNX Runtime inference, batching
    └── test/java/io/agentis/memory/
        ├── resp/RespDecoderTest.java
        ├── store/KvStoreTest.java
        └── integration/
            └── BasicCommandsTest.java      # Jedis against running server
```

The project is in **Layer 1 implementation phase**. TCP server, RESP protocol, and basic KV commands (SET, GET, PING) are fully implemented. Vector search and persistence (AOF/Snapshots) are planned for subsequent layers.

## Architecture

```
+------------------------------------------+
|          RESP Protocol Layer             |
|   TCP :6399, parses Redis commands       |
+------------------------------------------+
|          Command Router                  |
|   SET/GET/DEL/TTL  -> KV Store           |
|   MEMSAVE/MEMQUERY -> Vector Engine      |
+-------------------+----------------------+
|    KV Store       |   Vector Engine      |
|                   |                      |
| ConcurrentHashMap | Chunker (sentences)  |
| TTL / Expiry      | ONNX Embedding       |
| AOF + Snapshots   | HNSW Index (jvector) |
+-------------------+----------------------+
```

### Key Design Decisions

- **MEMSAVE is async:** KV write is synchronous (returns `+OK` immediately), chunking + embedding + HNSW indexation runs in background. Use `MEMSTATUS key` to check indexation status.
- **Namespace isolation by convention:** prefix before first `:` (e.g. `agent1:obs`). Not a security boundary — any authenticated client can read/write any namespace. True tenant isolation requires separate server instances, each with its own `--requirepass` and port. ACL-based namespace isolation is post-MVP; see the design spec's Security section.
- **Single HNSW index with post-filter:** namespace filtering over-fetches K×3 from HNSW, then filters. Fewer than K results is normal for sparse namespaces.
- **Memory accounting:** `--max-memory` governs KV value bytes only. Total RSS ≈ `--max-memory` + (chunk_count × 1.5KB) + 300MB baseline.

## Commands

### Standard Redis Commands
`SET`, `GET`, `DEL`, `EXISTS`, `EXPIRE`, `TTL`, `KEYS`, `SCAN`, `PING`, `QUIT`, `AUTH`, `INFO`, `DBSIZE`, `TYPE`, `CLIENT SETNAME/INFO`, `CONFIG GET`, `COMMAND`, `BGSAVE`

### Custom Commands
| Command | Description |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scrobot/agentis-memory](https://github.com/scrobot/agentis-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
