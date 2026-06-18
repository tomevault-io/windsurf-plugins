---
trigger: always_on
description: Local-first Rust CLI for durable per-repo project memory shared between Codex, Claude Code, and OpenCode. Treat [docs/reference/memhub-prd.md](docs/reference/memhub-prd.md) as the product authority and do not silently diverge from it.
---

# memhub — Codex CLI instructions

Local-first Rust CLI for durable per-repo project memory shared between Codex, Claude Code, and OpenCode. Treat [docs/reference/memhub-prd.md](docs/reference/memhub-prd.md) as the product authority and do not silently diverge from it.

This file is the Codex/OpenCode counterpart to `CLAUDE.md`. The two exist so Codex CLI, OpenCode CLI, and Claude Code sessions get the same orientation when they open this repo. Where they diverge it is intentional (different agent identifiers, different skill paths).

## Session Continuity

This repo is **memhub-primary** as of M7-002 (2026-05-13). The DB at `.memhub/project.sqlite` is the source of truth; rendered markdown is a local human-readable view under `.memhub/rendered/`. At session start, read `.memhub/rendered/PROJECT.md` if present for the "currently building / next up / open questions" state, the architecture narrative, and recent session notes; if it is missing, fall back to `memhub recall` / `memhub status`.

The mid-session routing rules — prefer recall over the `PROJECT_LEDGER.md` ledger, the turn-1-only PROJECT.md read, and re-render after changes — live in the memhub MCP server's own instructions (`src/mcp/mod.rs`) and are not duplicated here.

If recall returns a `warnings[].kind == "stale_embeddings"` entry, surface it and ask the user before invoking `/reindex`. Recall results stay usable in the meantime — the warning means hybrid scoring may be undercounting some rows, not that retrieval is broken.

The four legacy K9 files (`agent_docs/project_state.md`, `project_arch.md`, `project_decisions.md`, `project_backlog.md`) are historical archive — last accurate at commit `366cc1c`. Do not write to them; they are no longer authoritative. K9 integration is disabled in `.memhub/config.toml`.

## Cross-machine workflow

memhub state is **per-machine**. Each machine has its own `.memhub/project.sqlite`, its own embeddings, and its own rendered markdown under `.memhub/rendered/`. None of that is committed to git — only code, migrations, and the static tracked `CLAUDE.md` / `AGENTS.md` guardrails are.

**After `git pull` on a fresh or existing machine:**

```bash
cargo build --release
cargo run --release -- status   # first call auto-applies pending migrations from migrations/*.sql
```

`db::open_project` runs `migrations::apply_all` on every invocation; migrations are idempotent against `schema_migrations`, so no manual step is needed even if the schema bumped on another machine.

**To carry memory between machines (e.g. continue on Windows what you started on Mac):**

```bash
# on the source machine
memhub export ~/transfer/memhub-<repo>-<date>.json

# move the file via Drive / USB / scp — memhub itself stays offline

# on the target machine, with an existing memhub project
memhub import ~/transfer/memhub-<repo>-<date>.json          # refuses if target has data
memhub import ~/transfer/memhub-<repo>-<date>.json --force  # overwrite

# or to bootstrap a target that has no DB yet
memhub init --from-backup ~/transfer/memhub-<repo>-<date>.json
```

After import, the target's embeddings are not yet built (only the rows). Run `memhub index` to populate them — the import output prints this hint. Until then, recall falls back to FTS-only and may miss vector-similar matches.

If recall later surfaces a `stale_embeddings` warning (most likely after an embedding-model upgrade on either machine), follow the same rule as everywhere else: surface it and ask before invoking `/reindex`.

The export format is JSON v1, additive: older exports import cleanly into newer builds via `#[serde(default)]` on later-added fields. The format is defined in `src/export/v1.rs`.

**Config baseline travels with the repo.** The canonical defaults live in `.memhub/config.example.toml`, which is the **only** file inside `.memhub/` that is tracked by git. A fresh `memhub init` (or the first `open_project` call on a machine with no local config) copies the example verbatim into `.memhub/config.toml`. The local file stays gitignored and per-machine; edit the example to change the baseline for every machine. Fields that should not drift (deny_list, retrieval weights, render output dir, integrations) are documented at the top of the example as commit-back-here fields.

## Cross-machine Drive sync

Milestone 10 (design anchor: [docs/reference/memhub-prd-addendum-m10-drive-sync.md](docs/reference/memhub-prd-addendum-m10-drive-sync.md)) makes one user's repo memory follow them between their own machines through a synced folder, **without memhub ever going online**. It is the export/import flow above, automated and made fast-forward-aware.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kninetimmy/memhub](https://github.com/kninetimmy/memhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
