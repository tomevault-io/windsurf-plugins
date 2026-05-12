---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Rust Development

- This is primarily a Rust codebase. Ensure `cargo check` and `cargo clippy` pass after making changes.
- When modifying Rust code, follow idiomatic patterns: prefer `Result` over panics, use `?` operator for error propagation, and leverage the type system.

## Workflows

### Code Audit Workflow

- When working on audit items or code quality fixes, always complete ALL remaining items before ending. Track progress using a checklist format.
- After each fix, verify the fix compiles and doesn't introduce regressions by running `cargo check` and relevant tests.

## Task Completion

- When given a list of items to fix/verify, work through ALL items to completion. Do not stop mid-list.
- If a task is complex, break it into sub-tasks using TodoWrite but ensure every sub-task is resolved before finishing.

## Project Overview

**Detrix** is an LLM-first dynamic observability platform that enables developers and AI agents to add metrics to any line of code without redeployment — including code running in Docker containers and remote hosts (cloud debugging).

**Version:** 1.2.0
**Language:** Rust (edition 2021, rust-version 1.89)
**Architecture:** Clean Architecture with Domain-Driven Design

### Key Innovation

Leverages existing debuggers (debugpy, delve, lldb-dap) via **DAP (Debug Adapter Protocol)** to set **non-breaking observation points** (logpoints) that capture metrics without modifying source code or pausing execution.

## 🚨 CRITICAL: Use Detrix for Debugging

**NEVER add print(), console.log(), or logger.debug() statements for debugging.** Instead, use Detrix MCP to observe running processes directly without code modifications.

### ❌ DON'T Do This:
```python
# BAD - Don't add debug prints
print(f"DEBUG: user={user}, transaction={transaction}")
logger.debug(f"Amount: {amount}")
```

### ✅ DO This Instead:
```
"Add a metric to observe transaction.amount at payment.py line 127"
```

### Why Detrix Over Logging?
- Zero code changes - No source modifications or redeployment
- Zero downtime - Inspect running processes immediately
- Production-safe - Safe for live environments
- Auto-cleanup - Use TTL for temporary debugging
- Rich context - Stack traces + variable snapshots
- Clean codebase - No debug statement clutter

## Architecture

**Full details:** See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

### Clean Architecture Layers

```
┌────────────────────────────────────────────┐
│  detrix-cli, detrix-api    [Interface]    │
├────────────────────────────────────────────┤
│  detrix-application        [Application]   │
│  - Services (use cases)                    │
│  - Safety validation                       │
├────────────────────────────────────────────┤
│  detrix-ports              [Ports]         │
│  - Port traits (interfaces)                │
│  - Repository, adapter, cache, output      │
├────────────────────────────────────────────┤
│  detrix-storage, detrix-dap [Infrastructure]│
│  detrix-lsp, detrix-output                 │
│  - Implements port traits                  │
├────────────────────────────────────────────┤
│  detrix-core, detrix-config [Domain]       │
│  - Entities, value objects                 │
│  - Pure domain logic                       │
└────────────────────────────────────────────┘
```

### Dependency Rules (MUST FOLLOW)

```
detrix-cli        → can depend on ALL crates
detrix-api        → detrix-application, detrix-core, detrix-config
detrix-application→ detrix-ports, detrix-core, detrix-config ONLY
detrix-ports      → detrix-core, detrix-config ONLY
detrix-storage    → detrix-ports, detrix-core, detrix-application* (implements traits)
detrix-dap        → detrix-ports, detrix-core, detrix-application* (implements traits)
detrix-lsp        → detrix-ports, detrix-core, detrix-application* (implements traits)
detrix-output     → detrix-ports, detrix-core, detrix-application* (implements traits)
detrix-testing    → detrix-ports, detrix-core, detrix-application (test mocks)
detrix-core       → NOTHING (pure domain)
```

**Cross-cutting crates** (allowed as dependencies everywhere): `detrix-logging`. This is a thin tracing facade (macro re-exports + subscriber init) — a cross-cutting concern like config, not infrastructure.

**CRITICAL:** `detrix-ports` defines port traits. `detrix-application` NEVER depends on infrastructure crates like `detrix-storage` or `detrix-dap`. Infrastructure crates implement traits from `detrix-ports`.

**Note (*):** Infrastructure crates depend on `detrix-application` for shared types (JwksValidator, safety validators). This deviation is accepted - the key invariant (application never imports infrastructure) is maintained.

## Ubiquitous Language

Use these terms consistently:

| Term | Definition | NOT |
|------|------------|-----|
| **Metric** | An observation point in code | logpoint, probe, observation |
| **Expression** | Code to evaluate at the observation point; a metric can have multiple expressions | code snippet, query |
| **Location** | File path + line number (`@file.py#127`) | path, position |
| **Adapter** | DAP debug adapter for a language | debugger, connector |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flashus/detrix](https://github.com/flashus/detrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
