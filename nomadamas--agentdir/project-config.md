---
trigger: always_on
description: Read-only, agent-optimized navigation view for general-purpose file trees. agentdir restructures a virtual file tree over original files so AI agents, scripts, and humans can explore and navigate without touching the originals. The materialized view is read-only by design; when a consumer needs to edit, it resolves the original source path via `stat()`/`export_mapping()` and edits the original file directly.
---

# AGENTS.md — agentdir

## What This Project Is

Read-only, agent-optimized navigation view for general-purpose file trees. agentdir restructures a virtual file tree over original files so AI agents, scripts, and humans can explore and navigate without touching the originals. The materialized view is read-only by design; when a consumer needs to edit, it resolves the original source path via `stat()`/`export_mapping()` and edits the original file directly.
Rust workspace with two crates (`agentdir`, `agentdir-cli`) and Python bindings (`bindings/python`).

## Project Scope

agentdir is **infrastructure-level plumbing** — it is intentionally NOT an AI intelligence layer.

- **Read-only navigation view**: the materialized virtual tree is for exploration only, not editing. Materialized files are set read-only (0o444 on Unix, read-only attribute on Windows) to enforce this contract.
- **Edits go to the original file**: when a consumer needs to modify a file, it resolves the original source path via `stat()` or `export_mapping()` and edits the original file directly. The original file remains writable; the virtual tree never is.
- **Provides restructuring tools**: map, unmap, mv, cp, rename, mkdir, rmdir — enabling any consumer (AI agent, script, human) to reorganize a virtual file tree independently of the real source layout.
- **The restructuring agent is out of scope**: agentdir gives you the tools to restructure; it does not decide *what* to restructure or *why*. That intelligence lives in a separate repository. The "agent" in the name refers to the intended consumer, not something this project implements.
- **Targets all file types**: documents, spreadsheets, presentations, PDFs, images, media, datasets, plain text, binaries — any file the OS can stat. agentdir is file-format-agnostic.
- **No file parsing**: agentdir does not read, interpret, index, or transform file contents. It tracks whether a file has been created, modified, or deleted via metadata (mtime + size), and materializes copies via CoW reflinks.
- **Change tracking is the core value**: accurate, cross-platform detection of original-file mutations — additions, modifications, deletions — propagated to the virtual tree automatically.

## Out of Scope

The following are explicitly **not goals** of this project:

- AI/LLM integration, semantic understanding, or intelligent file routing
- File content parsing, full-text indexing, or search
- The orchestrator/agent that decides how to restructure the virtual tree
- File format conversion or transformation
- Dependency graph analysis, AST parsing, or language-aware features
- Access control, permissions, or multi-tenancy
- Editing files in the virtual tree — the materialized view is read-only by design; all edits are delegated to the original source path

## Module Map (`crates/agentdir/src/`)

| Module | Purpose |
|--------|---------|
| `lib.rs` | Module re-exports, `version()` |
| `types.rs` | `VirtualPath`, `SourcePath`, `ContentHash`, `CatalogEntry`, `EntryType`, `SourceMetadata`, `Manifest` |
| `error.rs` | `AgentdirError` enum via `thiserror` |
| `catalog.rs` | In-memory virtual filesystem catalog with O(1) lookup index |
| `materializer.rs` | Creates real files on disk via CoW reflinks or byte-copy fallback |
| `manifest.rs` | Atomic JSON persistence (write-tmp, fsync, rename) |
| `reflink.rs` | Safe wrapper around `reflink_copy::reflink_or_copy` |
| `backend/mod.rs` | `Backend` trait: scan, metadata, read_bytes, watch |
| `backend/local.rs` | `LocalBackend`: WalkDir scanning, `notify` watcher with debounce |
| `reconciler.rs` | Change detection: source events to `ChangeAction`s, full reconciliation |
| `workspace.rs` | Top-level API facade: init, open, map, unmap, refresh, mv, cp |
| `watcher.rs` | `FileWatcher` with debounced events + periodic polling fallback |

## CLI (`crates/agentdir-cli/src/main.rs`)

Commands: `init`, `map`, `unmap`, `status`, `refresh`, `mv`, `cp`, `mkdir`, `rmdir`, `watch`

## Python Bindings (`bindings/python/`)

PyO3 bindings exposing `Workspace` class with full API: `init`, `open`, `map`, `unmap`, `mv`, `cp`, `mkdir`, `rmdir`, `rename`, `exists`, `stat`, `read_bytes`, `refresh`, `status`, `export_mapping`, `map_batch`, `list_snapshots`, `destroy_snapshot`.

| Path | Purpose |
|------|---------|
| `src/lib.rs` | PyO3 `#[pymodule]` — wraps `agentdir::Workspace` |
| `python/agentdir/__init__.py` | Re-exports from native `_agentdir` module |
| `python/agentdir/_agentdir.pyi` | PEP 561 type stubs |
| `tests/` | 78 pytest tests covering all API methods |
| `pyproject.toml` | maturin build, uv deps, ruff + pytest config |

## Cross-Platform Notes

- **VirtualPath** always uses `/` internally on all platforms
  - `types.rs` skips `Component::Prefix` (Windows drive letters)
  - `virtual_path_for_relative()` normalizes via component iteration — never uses `display()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NomaDamas/agentdir](https://github.com/NomaDamas/agentdir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
