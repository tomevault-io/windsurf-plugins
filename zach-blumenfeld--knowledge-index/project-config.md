---
trigger: always_on
description: Instructions for AI agents (Claude, Codex, Cursor, etc.) working **on** the `knowledge-index` codebase. If you are an agent being asked by a user to *use* `ki` (index their notes, search their knowledge base, etc.), read `skills/knowledge-base/SKILL.md` instead.
---

# AGENTS.md

Instructions for AI agents (Claude, Codex, Cursor, etc.) working **on** the `knowledge-index` codebase. If you are an agent being asked by a user to *use* `ki` (index their notes, search their knowledge base, etc.), read `skills/knowledge-base/SKILL.md` instead.

## What this repo is

`knowledge-index` (CLI: `ki`) is a **search index** over a folder of markdown, backed by Neo4j — it syncs the filesystem into a knowledge graph and serves fast search / navigation / retrieval. The primitive verbs are `ki index` (sync) and `ki search` (query); read/navigation (`ki outline` [alias `ki tree`], `ki get`, `ki status`, `ki vault list`), index removal (`ki drop`, `ki nuke`), and setup (`ki configure`, `ki profile list`, `ki init`, `ki skill`) sit on top.

Canonical design lives in `docs/scoping.md` (profiles, vaults, config, the command surface, and the local/remote scoping model) and `docs/general-philosophy.md` (the tenets); the schema is `docs/data-model/schema.md`. `docs/README.md` indexes everything.

## Non-negotiable design principles

These constrain every change you make. If a proposed feature violates one of these, reject the proposal rather than working around it.

1. **`ki` is an index, not a document store.** Never mutate user-owned source files (`.md`). All `ki` output lives in `~/.config/ki/` (config), Neo4j (the index), or `.ki/vault.yaml` (vault identity + optional user-authored description per vault). `ki` writes the `uri:` field on first creation and otherwise touches user-authored fields **only when the user explicitly asks** via a flag (`ki index --description "..."` writes `description:`; without the flag, `ki` is read-only). No `--purge` flag, no "auto-organize my notes," no rewriting frontmatter. See `docs/general-philosophy.md`.
2. **The backend is opaque to the user.** From the user's and the agent-as-user's perspective, `ki` is a search tool. They don't need to know about Cypher, Neo4j, or graph traversal. Don't surface backend concepts (Cypher errors, node labels, etc.) in default output.
3. **One source of truth per concern.** Config lives at `~/.config/ki/config.yaml`; vault identity + per-vault user metadata live in `.ki/vault.yaml`; graph data lives in Neo4j. Don't introduce parallel state.
4. **Safe by default, dangerous by flag.** Destructive operations (whole-vault removal) require explicit flags AND typed confirmation. Cloud-resource creation (Aura) requires explicit consent even on agent auto-mode. See `docs/general-philosophy.md` (safe by default) and `skills/knowledge-base/SKILL.md` (agent auto-mode rules).

## Project map

| Path                              | What's there                                                                                                                                                            |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `src/ki/cli.py`                   | Click entry point. Top-level commands (`configure`, `index`, `search`, `get`, `outline` [alias `tree`], `status`, `drop`, `nuke`, `init`) + subcommand groups (`profile list`, `vault list`, `skill {list,install,remove,print}`). Each command lives in its own module under `src/ki/commands/`. |
| `src/ki/config.py`                | XDG-aware config loader; named profiles; 0600 mode on write; `KI_PROFILE` env-var override.                                                                             |
| `src/ki/profile_resolve.py`       | Resolve which profile a command uses: `--profile` → the vault's `.ki` binding → config default. `ki search` resolves stricter (see `docs/scoping.md`).                  |
| `src/ki/vault.py`                 | `.ki/vault.yaml` marker IO (slug `uri`, bound profile name, optional user-authored description); slug rules + `-N` collision; Folder/Document/Section URI construction.  |
| `src/ki/parser/markdown.py`       | markdown-it-py-based parser. Builds section tree per *Content Construction Rules* (Rule 1–3) and exposes a DFS-ordered flat list for `NEXT_SECTION`.                    |
| `src/ki/parser/frontmatter.py`    | python-frontmatter wrapper. Splits YAML frontmatter into `aliases`, `frontmatterCreatedAt`, and a JSON blob of unknown keys.                                            |
| `src/ki/ingest/pipeline.py`       | Per-vault orchestrator: schema, per-vault upsert, fileHash skip, concurrent reads (aiofiles), single Neo4j write session, one doc at a time, LINKS_TO post-pass.        |
| `src/ki/ingest/batcher.py`        | UNWIND batching + Neo4j-OOM auto-recovery (halve and retry once, continue smaller).                                                                                     |
| `src/ki/ingest/queries.py`        | Batched `UNWIND` ingest Cypher — matches `docs/data-model/ingest-cypher.md`.                                                                                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zach-blumenfeld/knowledge-index](https://github.com/zach-blumenfeld/knowledge-index) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
