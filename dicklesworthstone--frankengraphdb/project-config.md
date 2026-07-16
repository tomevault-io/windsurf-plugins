---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — frankengraphdb

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 — THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time.

---

## Branch Policy

- Primary branch is `main`.
- Do not reference `master` in docs/scripts.
- If release instructions require sync, push `main:master` after `main`.

---

## Project Mission

`frankengraphdb` is a **blank-slate, memory-safe, ultra-high-performance property-graph database in Rust**, built entirely on the Franken/asupersync ecosystem. It ships as one codebase in three postures: an **embedded library** (`fgdb`), a **server** (`fgdbd`) speaking the native FGP wire protocol plus HTTP/2, gRPC, WebSocket, and a Bolt-compat subset, and a **CLI** (`fgdb`). Larger-than-memory operation is first-class in all three.

The leapfrog is not one trick; it is the *composition* of six bets, each at or beyond the current frontier, made feasible only because the foundation libraries already exist:

- **B1 — One Version Universe.** MVCC versions, time-travel history, replication, change subscriptions, and git-style database branches are *the same mechanism*: an append-only, content-addressed, RaptorQ-coded commit stream (**Chronicle**).
- **B2 — Graph-Structured LSM ("Strata").** Adjacency lives in three temperature tiers (versioned delta blocks → sealed compressed CSR runs → archived anchors): transactional writes *and* static-CSR scan speed on one store.
- **B3 — Unified Factorized/WCO Execution ("Loom").** One Free-Join-style operator family subsumes binary hash joins, worst-case-optimal multiway joins, and factorized intermediates, running vectorized and morsel-parallel over Strata runs that *are already tries*.
- **B4 — Incremental Everything ("Ripple").** A DBSP-style Z-set delta algebra is the single engine for recursive queries, materialized views, standing queries/subscriptions, and incremental analytics — fed by the commit stream, which is *already* a Z-set stream.
- **B5 — Determinism as a Product Feature.** CGSE tie-break policies, complexity witnesses, and plan certificates make every result reproducible and auditable; every adaptive decision emits a replayable **decision card**; the whole database runs under asupersync's lab runtime for DPOR-explored, seed-replayable, chaos-injected testing.
- **B6 — Agent-Native by Construction.** Branch-per-agent isolation, capability-scoped (macaroon) subgraph authorization, provenance as first-class edges, and one hybrid vector+text+graph retrieval operator — purpose-built for GraphRAG and multi-agent memory.

**The single source of truth for what we are building and why is [`COMPREHENSIVE_PLAN_FOR_THE_DESIGN_OF_FRANKENGRAPHDB.md`](COMPREHENSIVE_PLAN_FOR_THE_DESIGN_OF_FRANKENGRAPHDB.md).** Read it before writing any subsystem.

### What we stand on (the closed dependency universe)

- `/dp/asupersync` — the operating system: structured-concurrency runtime (regions, obligations, `Cx` capability contexts, three-lane scheduler), the **lab runtime** (virtual time, DPOR, chaos, crashpacks), RaptorQ (RFC 6330), the full networking stack (TCP/UDP/QUIC/HTTP/1.1/2/3/WebSocket/TLS/gRPC), channels/combinators, Spork/OTP supervision, macaroons, metrics/OTel. Consumed as-is; we reimplement **none** of it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/frankengraphdb](https://github.com/Dicklesworthstone/frankengraphdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
