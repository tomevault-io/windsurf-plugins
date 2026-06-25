---
trigger: always_on
description: This document is a quick operational index for agents working on NovaRocks.
---

# NovaRocks - AI Agents Guide

This document is a quick operational index for agents working on NovaRocks.
It is designed to help you quickly:
- locate the right code paths
- understand the current execution architecture
- implement changes without semantic drift

This guide focuses on high-frequency implementation details and modification entry points.

---

## 1. Project Overview

NovaRocks is a **Rust-based, cloud-native, compute-storage decoupling friendly**
analytical query engine.

It now has two first-class modes:

- **StarRocks-compatible backend mode**
  - FE-compatible protocol behavior with zero FE awareness changes.
  - C++ Shim handles brpc access and protocol bridging.
  - Rust handles thrift plan lowering, pipeline execution, exchange, and connectors.

- **Standalone SQL engine mode**
  - Runs without StarRocks FE.
  - Provides a MySQL-compatible local server through `standalone-server`.
  - Owns SQL parsing, analysis, planning, codegen, catalog state, Iceberg catalog
    dispatch, managed-lake metadata, and SQL test execution.

Columnar processing is centered on Arrow `RecordBatch` / `Chunk` in both modes.

---

## 2. Architecture Overview (Current Code)

### 2.1 StarRocks-Compatible Backend Mode

```text
StarRocks FE
  |- HeartbeatService (Thrift) -------> Rust: src/service/heartbeat_service.rs
  |- BackendService (Thrift, be_port) -> Rust: src/service/backend_service.rs
  `- PInternalService (brpc) ---------> C++ Shim: src/shim/brpc_server.cpp
                                          |
                                          `- FFI (C ABI, compat.h)
                                               v
                                          Rust FFI: src/service/engine_ffi.rs
                                               v
                                          Query Execute: src/service/internal_service.rs
                                               v
                                          Lowering: src/lower/**
                                               v
                                          Pipeline: src/exec/pipeline/**
                                               v
                      +---------- ResultBuffer: src/runtime/result_buffer.rs
                      `---------- Exchange (gRPC): src/runtime/exchange.rs + src/service/grpc_*.rs
```

### 2.2 Standalone SQL Engine Mode

```text
SQL client / SQL test runner
  `- MySQL protocol -----------> Rust: src/server/mod.rs
                                      |
                                      v
                            Standalone Engine: src/engine/**
                                      |
                                      v
                     SQL Parser / Analyzer / Optimizer / Codegen:
                         src/sql/parser/**
                         src/sql/analyzer/**
                         src/sql/optimizer/**
                         src/sql/codegen/**
                                      |
                                      v
                            Pipeline / Runtime / Connectors
                                      |
                 +--------------------+--------------------+
                 |                    |                    |
          Local Parquet        Iceberg Catalogs        Managed Lake
        in-memory catalog    memory/hadoop/rest/hive   SQLite + object store
```

---

## 3. Non-Negotiable Rules (High Priority)

1. **Strictly follow FE-provided plan and type metadata**
   No fallback behavior, no guessed defaults, no implicit type downgrade.

2. **Fail fast on unsupported or ambiguous semantics**
   Return explicit errors in parsing/lowering stages instead of "best effort" execution.

3. **Keep protocol and execution responsibilities separated**
   C++ Shim is the protocol gateway; execution semantics belong to Rust.

4. **Keep FE-compatible and standalone responsibilities explicit**
   FE-compatible paths follow FE-provided thrift metadata. Standalone paths own
   SQL parsing, catalog resolution, and session context. Do not mix assumptions
   between the two modes without checking the active entrypoint.

5. **Distributed deployment is the source of truth; standalone is test-only**
   The real, user-facing deployment is NovaRocks distributed (1 NovaRocks FE +
   N BE; CI baseline 1FE+3BE). The single-process all-in-one form ("standalone")
   is only a testing convenience. Do NOT model for, or add special-case branches
   for, the single-process form. Cluster-topology quantities (broadcast fanout,
   backend count, per-node budgets) must be read dynamically from the live BE
   registry, never hardcoded or defaulted to "single-process = 1 node". Tests
   must never pass only in standalone while failing under 1FE+3BE.

6. **Language policy**
   - User interaction and design docs: Chinese
   - Code comments, logs, error messages, commit messages: English

---

## 4. Key Code Index (Validated Against Current Repository)

### 4.1 Entrypoints and Services

- `src/main.rs`
  Process entry. Dispatches FE-compatible modes (`run`, `start`, `stop`,
  `restart`) and the `standalone-server` mode.

- `src/server/mod.rs`
  MySQL-compatible standalone server, session context, SQL batch splitting,
  query timeout, and embedded statement routing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NovaRocks/NovaRocks](https://github.com/NovaRocks/NovaRocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
