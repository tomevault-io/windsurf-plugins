---
trigger: always_on
description: OxideSEO is an open-source, cross-platform desktop application for SEO site crawling and technical auditing. It competes with Screaming Frog, Sitebulb, and Netpeak Spider. The architecture is Tauri v2 (Rust backend) + React (TypeScript frontend), dual licensed MIT / Apache 2.0.
---

# CLAUDE.md — OxideSEO Development Guide

## What This Project Is

OxideSEO is an open-source, cross-platform desktop application for SEO site crawling and technical auditing. It competes with Screaming Frog, Sitebulb, and Netpeak Spider. The architecture is Tauri v2 (Rust backend) + React (TypeScript frontend), dual licensed MIT / Apache 2.0.

## Quick Reference

```bash
# Dev mode (frontend + Rust backend with hot reload)
npx tauri dev

# Production build
npx tauri build

# Rust tests
cd src-tauri && cargo test

# Rust lint
cd src-tauri && cargo clippy --all-targets -- -D warnings

# Rust format check
cd src-tauri && cargo fmt --all -- --check

# Frontend tests
npm run test

# Frontend lint
npm run lint

# Frontend type check
npm run typecheck

# Frontend format
npm run format:check

# E2E tests (Playwright against Vite dev server)
npm run test:e2e

# E2E tests interactive UI mode
npm run test:e2e:ui

# E2E tests debug mode (with inspector)
npm run test:e2e:debug

# Generate app icons (requires a 1024x1024 source PNG)
npx tauri icon app-icon.png
```

## Current State

**All 8 phases and 52 deliverables are complete.** The application is feature-complete per the development plan (`.claude/plans/seo-crawler-development-plan.pdf`). Deferred features D-1 through D-7 and D-9 are also implemented. Only D-8 (crawl scheduling) was intentionally skipped — it requires OS-native scheduling and headless CLI mode with poor effort-to-value ratio. See `.claude/plans/release-tasks.md` for remaining pre-release operational tasks (E2E tests, code signing, auto-update, crash reporting).

### Deferred Features (post-plan)

Beyond the 52 plan deliverables, these features were added:

- **D-1: Crawl deletion** — `delete_crawl` command with 9-table cascading delete, Dashboard dropdown with confirmation dialog
- **D-2: Crawl re-run** — `rerun_crawl` command clones config from DB, Dashboard dropdown action
- **D-3: Keyboard shortcuts** — Global hotkeys for navigation and actions
- **D-4: ResourceMeter** — Real-time memory RSS gauge + throughput stats in CrawlMonitor (raw FFI on macOS, `/proc/self/status` on Linux)
- **D-5: PDF export** — A4 summary report via `printpdf` crate
- **D-6: XLSX export** — Multi-sheet Excel with severity color-coding via `rust_xlsxwriter` crate
- **D-7: SiteTreeView** — Collapsible hierarchical URL tree in results explorer
- **D-9: Crawl comparison** — Cross-crawl diff with overview, page/issue/metadata diff tabs, Dashboard compare mode
- **D-8: Crawl scheduling** — NOT implemented (requires OS-native cron/Task Scheduler + headless mode)

## Architecture Invariants

These design decisions are intentional and should not be changed:

1. **Channel-based actor model** — The crawl engine uses `mpsc` channels between orchestrator, fetch workers, parse pool, and storage writer. Do not collapse these into a single loop.

2. **Rayon for CPU-bound work** — HTML parsing and rule evaluation happen on the rayon thread pool, NOT on tokio. Tokio is for async I/O only.

3. **Dedicated storage writer thread** — All SQLite writes go through a single dedicated thread to avoid WAL contention. Reads can happen from any thread via `Database::with_conn`.

4. **Batched transactions** — The storage writer accumulates 100-500 records before flushing in a single transaction. This is critical for write throughput.

5. **Server-side data operations** — Sorting, filtering, and pagination happen in Rust/SQLite via Tauri commands. The frontend NEVER holds the full dataset in memory.

6. **URL normalization before hashing** — Every URL must pass through `normalize_url()` before `hash_url()`. The blake3 hash is the dedup key.

7. **Manual redirect tracking** — reqwest redirect policy is set to `Policy::none()`. The fetcher follows redirects manually and records each hop.

8. **Progress event throttling** — `crawl://progress` events emit at most every 250ms or 50 URLs. Do not increase this frequency.

## Key Type Contracts

These types cross the IPC boundary. Changes must be synchronized between Rust and TypeScript:

| Rust (serde)             | TypeScript               | File                                     |
| ------------------------ | ------------------------ | ---------------------------------------- |
| `CrawlConfig`            | `CrawlConfig`            | `commands/crawl.rs` ↔ `types/index.ts`   |
| `CrawlStatus`            | `CrawlStatus`            | `commands/crawl.rs` ↔ `types/index.ts`   |
| `CrawlProgress`          | `CrawlProgress`          | `commands/crawl.rs` ↔ `types/index.ts`   |
| `PageRow`                | `PageRow`                | `storage/models.rs` ↔ `types/index.ts`   |
| `IssueRow`               | `IssueRow`               | `storage/models.rs` ↔ `types/index.ts`   |
| `LinkRow`                | `LinkRow`                | `storage/models.rs` ↔ `types/index.ts`   |
| `PaginatedResponse<T>`   | `PaginatedResponse<T>`   | `commands/results.rs` ↔ `types/index.ts` |
| `PluginInfo`             | `PluginInfo`             | `plugin/manager.rs` ↔ `types/index.ts`   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chris-c-thomas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
