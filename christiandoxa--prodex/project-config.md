---
trigger: always_on
description: This file applies to the entire repository.
---

# AGENTS.md

This file applies to the entire repository.

## Project Summary

`prodex` is a single-binary Rust CLI that wraps `codex` and manages multiple isolated `CODEX_HOME` profiles.

The codebase is now a Cargo workspace split across focused crates and modules:

- `src/main.rs`: binary entrypoint
- `src/lib.rs`: compatibility shim that includes `crates/prodex-app/src/lib.rs`
- `crates/prodex-app/`: application orchestration, command routing, Prodex-owned command handlers, profile flows, and runtime integration glue
- `crates/prodex-app-reports/`: reusable application report rendering helpers
- `crates/prodex-audit-log/`: audit log append, query, and rendering helpers
- `crates/prodex-bench-support/`: benchmark support helpers
- `crates/prodex-caveman-assets/`: embedded Codex/Claude Caveman plugin assets and Caveman home/config preparation
- `crates/prodex-cli/`: reusable CLI argument model, help text, and parse/default-run rewrite helpers
- `crates/prodex-codex-config/`: reusable Codex config parsing helpers
- `crates/prodex-context/`: context audit and compression helpers
- `crates/prodex-core/`: common path discovery and core filesystem helpers
- `crates/prodex-housekeeping/`: cleanup and duplicate-detection helpers
- `crates/prodex-profile-export/`: encrypted profile export/import envelope helpers
- `crates/prodex-profile-identity/`: account identity parsing and profile-name normalization helpers
- `crates/prodex-proxy-config/`: reusable upstream proxy/client configuration helpers
- `crates/prodex-quota/`: quota API models and quota classification helpers
- `crates/prodex-redaction/`: reusable log/diagnostic redaction helpers
- `crates/prodex-secret-store/`: reusable secret storage backend primitives
- `crates/prodex-runtime-anthropic/`: Anthropic compatibility translation helpers
- `crates/prodex-runtime-broker/`: side-effect-free runtime broker registry, health, and metrics DTOs
- `crates/prodex-runtime-broker-log/`: runtime broker log parsing and rendering helpers
- `crates/prodex-runtime-capabilities/`: runtime request compatibility surface detection
- `crates/prodex-runtime-claude/`: Claude Code runtime launch configuration helpers
- `crates/prodex-runtime-cookies/`: runtime proxy profile-scoped cookie relay helpers
- `crates/prodex-runtime-doctor/`: runtime diagnostics summary helpers
- `crates/prodex-runtime-launch/`: child process and runtime launch planning primitives
- `crates/prodex-runtime-log/`: runtime log path and marker helpers
- `crates/prodex-runtime-mem/`: runtime memory mode helpers
- `crates/prodex-runtime-metrics/`: runtime broker metrics model and Prometheus rendering
- `crates/prodex-runtime-policy/`: runtime policy parsing, validation, caching, and summary helpers
- `crates/prodex-runtime-proxy/`: side-effect-free runtime proxy boundary types, classifiers, path/log parsing, payload parsing, and transport helper logic
- `crates/prodex-runtime-quota/`: runtime quota adapter, snapshot, summary, and sort-key helpers
- `crates/prodex-runtime-state/`: runtime state, lane admission counters, and scheduled-save data structures
- `crates/prodex-runtime-store/`: runtime store merge and compaction helpers
- `crates/prodex-runtime-tuning/`: runtime tuning snapshot types, override parsing, and fault-injection counters
- `crates/prodex-session-store/`: persisted session metadata helpers
- `crates/prodex-shared-codex-fs/`: shared Codex home file operations
- `crates/prodex-shared-types/`: shared serializable command/runtime models used across modules
- `crates/prodex-state/`: state models and merge/compaction helpers
- `crates/prodex-terminal-ui/`: reusable terminal layout and printing helpers
- `crates/prodex-update-notice/`: npm/latest-version update notice cache and rendering helpers
- `README.md`: full user-facing documentation
- `QUICKSTART.md`: shorter installation and usage guide

## Core Principles

When changing `prodex`, keep these invariants intact:

1. The runtime proxy should be as transport-transparent as possible.
   - Let `codex` own reconnect, WebSocket fallback, and stream UX.
   - Do not invent new stream semantics unless strictly necessary.

2. Auto-rotate must remain built in to the proxy.
   - Profile/account selection is a `prodex` responsibility.
   - Transport behavior should remain as close as possible to upstream Codex.
   - Reliability improvements must not weaken affinity or allow mid-stream rotation.

3. Do not redefine upstream ChatGPT errors unless the proxy itself failed before any upstream response existed.
   - Prefer pass-through for upstream HTTP status, body, and stream payloads.

4. Do not print anything to the terminal while the Codex TUI is running.
   - Preflight output before launch is fine.
   - Runtime notices must go to log files, not stdout/stderr.

5. Repository prose must stay in English.

6. Runtime hot paths must stay non-blocking as much as possible.
   - Do not reintroduce disk I/O, broad file reads, or unbounded thread spawning into the request/stream hot path.
   - Prefer async transport and bounded background work over ad hoc blocking behavior.

7. Prodex-owned screens should be terminal-responsive.
   - Prefer adapting to the current terminal width instead of assuming a fixed 110-character layout.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christiandoxa/prodex](https://github.com/christiandoxa/prodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
