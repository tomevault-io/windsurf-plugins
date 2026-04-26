---
trigger: always_on
description: > This file is the primary instruction set for Claude Code when working on this project.
---

# CLAUDE.md — Git Flotilla

> This file is the primary instruction set for Claude Code when working on this project.
> Read this entire file before making any changes. Re-read relevant sections before each task.

---

## Project Identity

**Git Flotilla** is a cross-platform desktop GUI (Windows, macOS, Linux) for managing multiple GitHub and GitLab repositories at scale. It is built for DevOps engineers, agencies, and teams who manage many repos and need to perform batch operations, security patching, dependency auditing, and repo health monitoring from a single interface.

**Stack:** Rust · Tauri v2 · TypeScript · Vue 3 (Composition API) · Pinia · Tailwind CSS v4 · Vite

**Repo:** `git-flotilla`
**Crate name:** `git-flotilla`
**Binary name:** `git-flotilla`

---

## Tech Stack Rules

### Rust / Tauri Backend
- All filesystem access, GitHub/GitLab API calls, Git operations, and CVE scraping happen in **Rust only** — never in the frontend
- Use `tauri::command` to expose backend functions to the frontend
- Use `tokio` for async — all I/O must be non-blocking
- Use `reqwest` for HTTP (GitHub API, GitLab API, CVE feeds)
- Use `serde` / `serde_json` for all serialisation
- Use `sqlx` with **SQLite** (via Tauri's app data directory) for local persistence of scan results, repo configs, CVE cache, and audit logs
- Use `git2` (libgit2 bindings) for local Git operations where needed
- Error handling: always use `Result<T, E>` — no `unwrap()` in production paths; propagate errors to frontend via Tauri's error serialisation
- Secrets (tokens, credentials) must be stored using `keyring` crate (OS keychain), never in plaintext config files

### Vue 3 / TypeScript Frontend
- Use **Composition API** exclusively (`<script setup lang="ts">`) — no Options API
- Use **Pinia** for all global state — one store per domain (repos, scans, CVEs, settings, auth, operations)
- Use **Vue Router** for navigation between views
- All Tauri backend calls go through a typed wrapper layer in `src/services/` — never call `invoke` directly in components
- Use **Tailwind CSS v4** utility classes — no custom CSS except for design tokens in `tailwind.config.ts`
- Component library: build internal components in `src/components/ui/` before reaching for external libraries
- Forms: use `vee-validate` + `zod` for all form validation
- Use `@tanstack/vue-query` for async state management of backend calls (loading, error, refetch)

### General
- TypeScript strict mode is **always on** — no `any` types, no `@ts-ignore`
- All new features require a corresponding entry in `PLANNING.md` marked as `[implemented]`
- Run `cargo clippy` and `cargo fmt` before committing Rust code
- Run `pnpm lint` and `pnpm typecheck` before committing frontend code

---

## Project Structure

```
git-flotilla/
├── src-tauri/                  # Rust / Tauri backend
│   ├── src/
│   │   ├── main.rs             # Tauri app entry point
│   │   ├── lib.rs              # Library root
│   │   ├── commands/           # Tauri command handlers (one file per domain)
│   │   │   ├── auth.rs         # GitHub/GitLab auth commands
│   │   │   ├── repos.rs        # Repo discovery, listing, repo list management
│   │   │   ├── scan.rs         # Repo scanning (versions, package managers, files)
│   │   │   ├── packages.rs     # Dependency matrix, overlap analysis, changelog
│   │   │   ├── cve.rs          # CVE scraping, alerts, watchlist, incident timeline
│   │   │   ├── operations.rs   # Batch file updates, commits, PRs, pin/bump
│   │   │   ├── merge_queue.rs  # PR merge queue, batch merge
│   │   │   ├── scripts.rs      # Custom script runner
│   │   │   ├── compliance.rs   # Secret scanning, licence compliance, branch protection
│   │   │   └── settings.rs     # App settings
│   │   ├── models/             # Serde structs matching frontend types
│   │   ├── db/                 # SQLx queries and migrations
│   │   │   └── migrations/     # SQL migration files
│   │   ├── services/           # Business logic (not Tauri commands)
│   │   │   ├── github.rs       # GitHub API client
│   │   │   ├── gitlab.rs       # GitLab API client
│   │   │   ├── cve_scraper.rs  # CVE feed polling service
│   │   │   ├── scanner.rs      # Repo scanning logic
│   │   │   ├── patcher.rs      # Package pin/bump logic, monorepo-aware patching
│   │   │   ├── changelog.rs    # Changelog fetching from GitHub Releases / CHANGELOG.md
│   │   │   ├── rate_limiter.rs # API rate limit tracking and auto-pause
│   │   │   ├── script_runner.rs # Custom script execution across repos
│   │   │   ├── secret_scanner.rs # Secret exposure detection (regex patterns)
│   │   │   ├── licence_checker.rs # Transitive dependency licence analysis
│   │   │   └── scheduler.rs    # Background job scheduler (hourly CVE, scheduled scans)
│   │   └── error.rs            # Unified error type
│   ├── Cargo.toml
│   └── tauri.conf.json
│
├── src/                        # Vue 3 / TypeScript frontend
│   ├── main.ts
│   ├── App.vue
│   ├── router/
│   │   └── index.ts
│   ├── stores/                 # Pinia stores
│   │   ├── auth.ts
│   │   ├── repos.ts
│   │   ├── repoLists.ts
│   │   ├── scans.ts
│   │   ├── packages.ts
│   │   ├── cve.ts
│   │   ├── operations.ts
│   │   ├── mergeQueue.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/immersedone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
