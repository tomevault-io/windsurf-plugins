---
trigger: always_on
description: `agentpack` is a Rust CLI that pins **GitHub-hosted skills** and **plugin directories** (`.claude-plugin` and/or `.cursor-plugin`) for a project.
---

# agentpack

`agentpack` is a Rust CLI that pins **GitHub-hosted skills** and **plugin directories** (`.claude-plugin` and/or `.cursor-plugin`) for a project.

**Source of truth for what to install** is **`agentpack.toml`** at the repo root (direct dependencies, project-local modes, and MCP settings). **`pack.lock`** (v2) lists every resolved **package** (direct and transitive from nested `agentpack.toml` files inside dependencies) with pinned commits and `cache_key`s. Both files live in the **project repo**.

All downloaded trees, the RedDB index, and your **`local/`** mirror live under a **user-wide agentpack home** (see below)—not under a repo-local `.agentpack/` directory. Staging for harnesses still uses a **per-project temp** directory (or **`AGENTPACK_STAGING_ROOT`**).

### Pre-release

**No backwards compatibility.** `agentpack` is pre-release: CLI behavior, lockfile shape, staging layout, env vars, and defaults may change without a migration period or deprecation window. Assume **breaking changes** between versions until a stable release is declared.

### Code structure (contributor rule)

**Per-harness code lives in `src/harness/<name>/`. Shared code lives in its subsystem. No re-export shims, no orphaned glue. No single-file folders except the uniform harness folders, no <40-line orphan modules.**

- **One folder per harness, uniformly.** Each coding agent is a folder **`src/harness/<name>/`** with `mod.rs` (+ submodules as needed) implementing the `Harness` trait — even small harnesses (`grok/`, `agy/`) are folders, so the layout is identical for all six. **Everything used by only that one harness lives there** — seed, attribution writer, MCP-config writer (including codex/grok's own copy of the `[mcp_servers]` TOML writer), hook renderer + support fn, credential bridging (`codex/auth.rs`), fake-home (`cursor/fake_home.rs`), workspace overlays, and `launch_command`. To understand or add a harness you read/create **one folder**, nothing else. The only files at `harness/` root are the shared harness *system*: `mod.rs` (trait + registry + contexts), `target.rs` (`HarnessTarget`), `launch.rs` (launch dispatch).
- **Shared infrastructure stays in its subsystem and never holds per-harness logic:** `staging/` = cross-harness staging passes (`pack_overlay`, `collision`, `guidance` collection, `dot_agents`, `mcp` collect+merge, `pipeline`) plus the shared `keep_attribution`/`NO_ATTRIBUTION_BODY` primitives; `hooks/` = the hook engine (`ir`, `collect`, `parse`, `stage`, `render` trait + `SupportLevel`, `runtime/{bridge,dispatch,handlers,output}`); `artifacts/` = artifact rendering; `fs_util.rs` = generic file-tree helpers (`copy_selected_entries`, `copy_merge_tree`). A file named after a harness (`cursor.rs`, `parse/claude.rs`) sitting in a shared subsystem is a smell — colocate it or, if it's actually a shared format adapter, name it for the format not the harness.
- **No single-file folders, no tiny orphans.** A module is a flat `foo.rs` unless it has real internal structure (the uniform harness folders are the one deliberate exception). A <40-line helper used by ≥2 modules lives *in* its natural home (its trait file, its caller, or `fs_util`), not in a file of its own. API-surface `pub use` in a subsystem `mod.rs` (e.g. `sync`, `github`) is a module boundary, not a shim — those stay.
- **One canonical home per type; no re-export shims.** `HarnessTarget` is defined only in `src/harness/target.rs` and imported everywhere as **`crate::harness::HarnessTarget`** — there are no `pub use …::HarnessTarget` chains through `artifacts`/`staging`/`sync`/`hooks::ir`. Don't add a re-export to "keep an old path resolving"; repoint the callers.
- **No orphaned glue modules.** Launch dispatch + shared launch helpers live in `harness/launch.rs` (called via `harness::launch`), not a separate top-level `launcher/`. If a module exists only to forward to another, delete it and inline the call.
- The trait owns *per-harness divergence*; genuinely cross-harness passes that take "all roots at once" (`pack_overlay`, `guidance`, `collision`) stay shared loops. Adding a 7th harness should mean: one new `src/harness/<name>/` + one line in `harness::all()` + one `HarnessTarget` variant + one `cli` subcommand.

### User data layout (`AGENTPACK_HOME`)

| Path | Purpose |
| --- | --- |
| **`$AGENTPACK_HOME/cache/<cache_key>/`** | Content-addressed package trees (GitHub tarball, or **copies** from filesystem / `local/`). |
| **`$AGENTPACK_HOME/cache/db.reddb`** | Metadata + alias map for fast repeat **`add`**, plus cached GitHub ref/tag lookups to reduce API calls. |
| **`$AGENTPACK_HOME/local/<owner>/<repo>/…`** | Optional offline mirror; same slash layout as **`owner/repo/…`** specs. |
| **`$AGENTPACK_HOME/projects/<hash>/cursor-overlay.manifest`** | Per-project Cursor overlay bookkeeping (not stored in the repo). |
| **`$AGENTPACK_HOME/projects/<hash>/agy-overlay.manifest`** | Per-project Antigravity workspace plugin overlay bookkeeping (not stored in the repo). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OlegHQ/agentpack](https://github.com/OlegHQ/agentpack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
