---
trigger: always_on
description: This document provides guidance for AI coding agents working with the Beam HTTP client codebase.
---

# AGENTS.md

This document provides guidance for AI coding agents working with the Beam HTTP client codebase.

Keep all LLM replies concise and accurate. Prefer the short response that fully answers the user's request. Clarify with user if anything no clear.

## Project Overview

Beam is a fast, lightweight HTTP client built with Rust and the gpui GUI framework. It provides features similar to Postman or Insomnia, including multi-workspace support, environment variables, authentication methods, and post-request scripting.

The hierarchy is: **Workspace → Folders → Requests**. Collections have been removed. Only global environments exist (no collection-scoped environments).

## Supported Features

// TODO

## Architecture

### Core Technologies

- **Language**: Rust 1.70+
- **GPUI Framework**: gpui
- **HTTP Client**:
- **Storage**: File-based persistence using TOML format
- **Scripting**: JavaScript execution for post-request scripts (via `boa` engine)

### Project Structure

```
src/
├── main.rs                  # Application entry point — bootstraps storage, initializes workspace, launches UI
├── lib.rs                   # Crate root — declares all public modules
├── app_shell.rs             # App-level state management, data-sync worker, layout state, startup preload
├── ui.rs                    # GPUI views and rendering — the main GUI layer (panels, editors, menus, etc.)
├── workspace_tree.rs        # Pure domain model — in-memory tree (SharedStore, Node, NodeKind, manifests)
├── models.rs                # Serializable DTOs — RequestFile, EnvironmentFile, WorkspaceFile, WorkspacesRegistryFile, LocalStateFile, etc.
├── request_authoring.rs     # Request authoring state — tabs, send-button logic, validation helpers
├── script.rs                # Post-request script execution — QuickJS runtime, console capture, test results
├── schema.rs                # Schema versioning — SCHEMA_VERSION_V1/V3, SchemaKind, version validation
├── paths.rs                 # File-system path definitions — DataRootPaths, BeamPaths, slugify
├── error.rs                 # Error types — BeamError enum and Result<T> alias
├── assets.rs                # Asset helpers — embedded theme contents, icon paths
└── storage/
    ├── mod.rs               # Storage DTOs + WorkspaceStorage trait — CRUD input structs, BootstrapReport
    ├── io_backend.rs        # StorageIoBackend trait — abstract I/O (read/write TOML, dirs, rename, remove)
    ├── workspace_repo.rs    # WorkspaceRepository — primary repository, all CRUD operations on SharedStore
    ├── registry_repo.rs     # RegistryRepository — loads/saves workspaces.toml, bootstraps default workspace
    └── fs_backend.rs        # FileSystemStorage — concrete std::fs adapter implementing StorageIoBackend
```

### Key Components

| Module / File | Role |
|---|---|
| `workspace_tree.rs` | **Pure domain model**. Holds `SharedStore` (in-memory tree), `Node`/`NodeKind` (`Folder` \| `Request`), manifest structs, and tree-manipulation helpers (name scoping, uniqueness checks, move/reorder logic). No I/O. |
| `models.rs` | **Serializable data structures**. Every TOML-backed entity (requests, environments, workspace, workspaces registry, local state) is defined here. Used by both the domain layer and the storage layer. |
| `storage/mod.rs` | **Storage contracts & DTOs**. Defines the `WorkspaceStorage` trait and all input structs consumed by repository methods (`CreateRequestInput`, `MoveFolderInput`, etc.). Also holds `BootstrapReport`. Parent refs (`RequestParentRef`, `FolderParentRef`) use `folder_id: Option<Ulid>` — `None` means workspace root. |
| `storage/io_backend.rs` | **I/O abstraction**. The `StorageIoBackend` trait decouples repository logic from the file system so tests can swap in a fake backend. |
| `storage/fs_backend.rs` | **Concrete file-system adapter**. `FileSystemStorage` implements `StorageIoBackend` using `std::fs`. Handles TOML serialization, atomic writes, and path-based operations. |
| `storage/workspace_repo.rs` | **Primary repository**. `WorkspaceRepository<B: StorageIoBackend>` loads the full workspace into `SharedStore`, then performs all CRUD (create, rename, move, delete, duplicate, reorder) while keeping disk and in-memory state in sync. |
| `storage/registry_repo.rs` | **Workspace registry**. `RegistryRepository` loads/saves `workspaces.toml`, bootstraps a default workspace on first run, and manages multi-workspace CRUD (create, delete, rename, switch). |
| `app_shell.rs` | **Application shell & orchestration**. Owns `AppShellState`, `DataSyncRuntime`, pane-split layout, startup preload logic, and the background command queue that feeds the repository. Manages workspace switching. |
| `ui.rs` | **GPUI front-end**. All view rendering, event handling, context menus, modal dialogs, workspace picker, and user-interaction logic lives here. |
| `request_authoring.rs` | **Request editor state**. Tab enums, send-button states, and validation helpers for the request authoring panel. |
| `script.rs` | **Script engine**. Executes post-request JavaScript via `rquickjs`, captures console output, and returns `ScriptExecutionResult`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hlcfan/beam](https://github.com/hlcfan/beam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
