---
trigger: always_on
description: These instructions apply to all AI coding agents working in this repository.
---

# Repository Agent Instructions

These instructions apply to all AI coding agents working in this repository.

## Skills

- **graphify** (`~/.claude/skills/graphify/SKILL.md`) — any input (code, docs, papers, images) → knowledge graph. Trigger: `/graphify`

When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.

## Architecture: Rust Backend Is the Source of Truth

The dashboard web app must not become a second backend. TrustLoopGuard has one durable/runtime backend: the Rust service in `crates/tl-server`, backed by `crates/tl-storage`. The Next.js app in `apps/web` is a UI and same-origin proxy layer only.

Authoritative backend ownership:
- Rust HTTP API: `crates/tl-server`
- Rust durable storage: `crates/tl-storage`
- Rust wire contracts: `crates/tl-core`
- Web dashboard/proxy: `apps/web`

Target architecture:

```text
                         source of truth
                               |
                               v
+---------+        +----------------------+        +----------------------+
| Browser | -----> | Next.js dashboard UI | -----> | Next API proxy      |
|         |        | apps/web pages       |        | apps/web/app/api/*  |
+---------+        +----------------------+        +----------+-----------+
                                                              |
                                                              | HTTP /v1/*
                                                              v
                                                   +----------+-----------+
                                                   | Rust API server     |
                                                   | crates/tl-server    |
                                                   +----------+-----------+
                                                              |
                                  +---------------------------+---------------------------+
                                  |                           |                           |
                                  v                           v                           v
                       +----------+-----------+    +----------+-----------+    +----------+-----------+
                       | Runtime guard logic  |    | Durable storage     |    | Wire contracts      |
                       | policy evaluation    |    | crates/tl-storage   |    | crates/tl-core      |
                       +----------------------+    +----------+-----------+    +----------------------+
                                                              |
                                                              v
                                                   +----------+-----------+
                                                   | Postgres / runtime  |
                                                   | persistence         |
                                                   +----------------------+

Forbidden source-of-truth path for guardrail/runtime data:

+----------------------+        +----------------------+
| Next API route       | -X---> | Web DB / Drizzle    |
| apps/web/app/api/*   |        | runtime ownership   |
+----------------------+        +----------------------+
```

SDK runtime integration path:

```text
Customer / integrator runtime

+----------------------+        +----------------------+
| Customer AI agent    | -----> | TrustLoopGuard SDK   |
| app code             |        | TS / Python / Rust   |
+----------------------+        +----------+-----------+
                                           |
                                           | POST /v1/check
                                           | Authorization: Bearer <api key>
                                           v
                                +----------+-----------+
                                | Rust API server     |
                                | crates/tl-server    |
                                +----------+-----------+
                                           |
                         +-----------------+-----------------+
                         |                 |                 |
                         v                 v                 v
              +----------+------+  +-------+----------+  +---+---------------+
              | Policy runtime  |  | Agent profiles   |  | Knowledge/runtime |
              | tl-engine       |  | tl-storage       |  | context           |
              +----------+------+  +------------------+  +-------------------+
                         |
                         v
              +----------+-----------+
              | Decision             |
              | allow/block/rewrite  |
              | escalate + trace_id  |
              +----------+-----------+
                         |
                         v
+----------------------+        +----------------------+
| TrustLoopGuard SDK   | -----> | Customer AI agent    |
| typed response       |        | applies decision     |
+----------------------+        +----------------------+

Trace persistence side effect:

+----------------------+        +----------------------+
| Rust API / engine    | -----> | Rust trace writer    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ducnguyen67201/TrustLoopGuard](https://github.com/ducnguyen67201/TrustLoopGuard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
