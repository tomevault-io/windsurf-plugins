---
trigger: always_on
description: A desktop disk usage visualizer — think "Disk Inventory X with a cleanup workflow." Scans a volume, renders contents as a squarified treemap, sunburst diagram, or flat table — colored by file type or age — and lets you queue files for Trash/permanent deletion. Built with **Tauri v2** (Rust backend + React/TypeScript frontend). Cross-platform: macOS, Linux, Windows.
---

# CLAUDE.md

## Project: Ansel

A desktop disk usage visualizer — think "Disk Inventory X with a cleanup workflow." Scans a volume, renders contents as a squarified treemap, sunburst diagram, or flat table — colored by file type or age — and lets you queue files for Trash/permanent deletion. Built with **Tauri v2** (Rust backend + React/TypeScript frontend). Cross-platform: macOS, Linux, Windows.

## Git workflow (IMPORTANT)

**Direct push to `main` is blocked.** `main` has branch protection enabled:

- **Require a pull request before merging** — all changes must go through a feature branch → PR → merge
- **Require status checks to pass** — CodeQL must succeed before merge
- **Only squash merges** are allowed (default merge method is squash)
- **No force pushes, no deletions** on `main`
- **Admins included** — no bypass allowed

**How to land changes:**

```bash
git checkout -b <branch-name>
# make changes, commit
git push -u origin <branch-name>
# Create a PR on GitHub
# Self-approve the PR (review → approve)
# Wait for CodeQL to pass (~2 min)
# Squash & merge
```

**CRITICAL: Before pushing to any branch, verify the associated PR is not already merged.** Run `gh pr view <branch> --json state` or check the PR URL. If the PR is merged, create a new branch from `main` (after `git pull`) instead of pushing to the old one.

## Supply-chain / security

- **OpenSSF Scorecard** — runs automatically on every push to `main` (`.github/workflows/scorecard.yml`)
- **CodeQL SAST** — runs on push to `main`, PRs into `main`, and weekly (`.github/workflows/codeql.yml`)
- **Dependabot** — enabled for automated dependency updates (`.github/dependabot.yml`)
- **Build provenance** — release workflow attests build provenance via `actions/attest-build-provenance`
- **Security policy** — [`SECURITY.md`](SECURITY.md) with reporting instructions
- **All GitHub Actions pinned by commit hash** (not version tags)
- **Top-level `permissions: read-all`** on all workflows; write permissions only at job level where needed

## Build & Dev

```bash
pnpm install          # first time only
pnpm tauri dev        # hot-reload frontend, incremental Rust rebuilds
pnpm tauri build      # production build → src-tauri/target/release/bundle/
```

- Requires: **Rust stable**, **Node 20+**, **pnpm**
- Frontend-only dev (without Tauri): `pnpm dev` (Vite only, no Rust backend)
- First cold-compile of Rust backend: ~30–60s on Apple Silicon

## Architecture

### Two halves

| Layer | Path | Language | Role |
|-------|------|----------|------|
| Backend | `src-tauri/` | Rust | File scanning (rayon-parallel), tree mutation, Tauri IPC commands |
| Frontend | `src/` | TypeScript + React 18 | UI, treemap rendering, state management, IPC calls via `@tauri-apps/api` |

### Backend (`src-tauri/src/`)

- **`lib.rs`** — Tauri app setup, all `#[tauri::command]` handlers, shared `AppState` (holds the scan tree in an `Arc<RwLock<Option<Node>>>`). Handlers: `start_scan`, `cancel_scan`, `get_node`, `get_subtree`, `list_children`, `top_files`, `extension_breakdown`, `move_to_trash`, `delete_paths`, `delete_path`, `show_in_file_manager`, `list_volumes`, `get_volume`, `home_dir`, `search`. Volume listing includes `total_bytes`/`free_bytes` via `disk_space()` (statvfs on Unix, GetDiskFreeSpaceExW on Windows). `get_volume` re-queries disk space for a single path (used to refresh after scan). Uses `tauri-plugin-opener` for file manager reveal.
- **`scanner.rs`** — The scan engine. `scan()` recursively walks the filesystem using **rayon parallel iteration** over directory entries. Nodes accumulate `size` (on-disk blocks × 512 on Unix, not apparent size), `file_count`, `mtime`, and `atime`. Directory `mtime`/`atime` are propagated as max-of-children up the tree. Helper fns: `find()`, `top_files()`, `extension_breakdown()`, `is_protected_path()` (prevents deletion of system directories like `/System`, `/Library`, `C:\Windows`, etc.).
- **`main.rs`** — Just calls `lib::run()`.

Key types:
- `Node` — full tree node (children inline), not sent over IPC (children skipped in serialization)
- `NodeSummary` — lightweight node sent to frontend (has `child_count` but no `children`)
- `SubtreeNode` — a subtree for treemap rendering, with `children` included
- `AppState` — `root: Arc<RwLock<Option<Node>>>` + `cancel: Mutex<Option<Arc<AtomicBool>>>`
- Uses **rayon global thread pool** (configured in `setup()` to `num_cpus - 2`, min 2 threads)

### Frontend (`src/`)

- **`App.tsx`** — Layout shell: three-column layout (sidebar, center, rightbar), toolbar controls. Manages side effects: Tauri event listeners, keyboard shortcuts, welcome→main cross-fade, sidebar resize. **State lives in the zustand store, not here.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fosterdill/ansel](https://github.com/fosterdill/ansel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
