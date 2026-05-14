---
trigger: always_on
description: > **Purpose:** Canonical reference for codebase patterns, architectural decisions, and mandatory conventions.
---

# NNTP Proxy — Development Guidelines

> **Purpose:** Canonical reference for codebase patterns, architectural decisions, and mandatory conventions.
> **Audience:** AI assistants (Claude Code), human contributors, code reviewers.
> **Principle:** Reuse > Reimplementation. When a pattern exists, use it — don't duplicate it.

---

## Architectural Overview

```
┌─────────────────────────────────────────────────────┐
│                 NNTP Proxy Server                   │
├─────────────────────────────────────────────────────┤
│ Runtime: tokio (async) + jemalloc (allocator)       │
│ Protocol: RFC 3977 NNTP + TLS (rustls)              │
└─────────────────────────────────────────────────────┘
           │
           ├──> Routing Layer (3 modes: Stateful, PerCommand, Hybrid)
           │    └─> BackendSelector: round-robin + health-aware
           │
           ├──> Connection Pool (deadpool)
           │    ├─> Pre-authenticated backend connections
           │    └─> Connection prewarming on startup
           │
           ├──> Caching Layer (2-tier)
           │    ├─> Memory cache: moka (LRU) or foyer-memory
           │    └─> Disk cache: foyer hybrid (psync I/O)
           │
           ├──> Buffer Management
           │    ├─> PooledBuffer [acquire()]: single-read I/O scratch (724KB, pre-faulted)
           │    └─> PooledBuffer [acquire_capture()]: accumulator for caching (768KB, pre-faulted)
           │
           └──> Pipeline Engine
                ├─> BackendQueue: batched ARTICLE/BODY requests
                └─> Batching: 4-16 commands per round-trip
```

### Routing Modes

1. **Stateful** (1:1 client↔backend mapping) — GROUP, XOVER, article-by-number commands
2. **PerCommand** (stateless, shared pool) — message-ID based operations only
3. **Hybrid** (default) — starts PerCommand, auto-switches to Stateful on GROUP/XOVER/etc.

**Important:** `run_stateful_proxy_loop` takes generic `W: AsyncWrite + Unpin` — **MUST** call `.flush().await?` after auth responses and backend writes. `handle_per_command_routing` uses concrete `WriteHalf<'_>` — changing to generic cascades through many signatures.

---

## Mandatory Patterns

### 1. Terminator Detection

**`TailBuffer` is the ONE AND ONLY place terminator detection can exist in this codebase. There must never be any other implementation anywhere.**

An NNTP multiline response ends with `\r\n.\r\n` (5 bytes). Detecting this correctly across async chunk reads is subtle and easy to get wrong. Getting it wrong causes **connection pool collapse**.

#### Forbidden Patterns — Never Write These

```rust
// ❌ ends_with check on accumulated buffer
if data.ends_with(b"\r\n.\r\n") { ... }
if data.ends_with(b".\r\n") { ... }

// ❌ Any helper wrapping ends_with
fn has_terminator(data: &[u8]) -> bool { data.ends_with(b"\r\n.\r\n") }

// ❌ Constant replicated outside tail_buffer.rs
const MULTILINE_TERMINATOR: &[u8] = b"\r\n.\r\n";

// ❌ Accumulate-then-check loop (the worst pattern)
let mut accumulated = Vec::new();
loop {
    accumulated.extend_from_slice(&buf[..n]);
    if accumulated.ends_with(b"\r\n.\r\n") { break; } // WRONG
}
```

#### Correct: Streaming (most common case)

```rust
use crate::session::streaming::tail_buffer::{TailBuffer, TerminatorStatus};

let mut tail = TailBuffer::default();
loop {
    let n = stream.read(buffer.as_mut_slice()).await?;
    if n == 0 { break; }
    let chunk = &buffer[..n];

    match tail.detect_terminator(chunk) {
        TerminatorStatus::FoundAt(pos) => {
            // CRITICAL: pos is the byte offset immediately AFTER "\r\n.\r\n"
            // chunk[..pos] includes the terminator and no bytes from the next response
            client.write_all(&chunk[..pos]).await?;
            break;
        }
        TerminatorStatus::NotFound => {
            client.write_all(chunk).await?;
            tail.update(chunk);
        }
    }
}
```

#### Correct: Complete-buffer validation

```rust
// TailBuffer with no prior state, checking a fully-accumulated buffer
TailBuffer::default().detect_terminator(&buffer).is_found()
```

#### Why `ends_with()` Causes Connection Pool Collapse

This caused a production bug (20 connections removed in rapid succession, pool exhausted):

1. Backend sends article body + start of next pipelined response in the same TCP segment
2. `read()` returns `[...article data...\r\n.\r\n200 Article follows\r\n...]`
3. `ends_with(b"\r\n.\r\n")` returns **false** — buffer ends with `...200 Article follows\r\n...`
4. Loop continues, consuming the `220 0 <next-msg-id>\r\n` for the next pipelined command
5. Next handler reads garbage → `Invalid` → `remove_with_cooldown()` → cascade to zero

`TailBuffer::detect_terminator()` returns `FoundAt(pos)` at the correct boundary — pipelined bytes are never consumed from the wrong context.

**Location:** `src/session/streaming/tail_buffer.rs` (30+ tests, production-proven)

---

### 2. I/O Buffer Management

`BufferPool` has **two distinct modes**. Using the wrong one causes silent reallocation and/or corrupted data.

#### Mode 1: Scratch buffers — `acquire()`

Pre-allocated fixed-size `Vec<u8>` with `len == capacity` (724KB, page-faulted). For single read operations.

```rust
// ✅ ALWAYS use for socket reads:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mjc/nntp-proxy](https://github.com/mjc/nntp-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
