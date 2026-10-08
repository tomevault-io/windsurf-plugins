---
trigger: always_on
description: Polylogue is a local, single-writer archive of AI coding and chat sessions
---

# Polylogue

Polylogue is a local, single-writer archive of AI coding and chat sessions
(Claude web and Code, ChatGPT, Codex, Gemini and Drive, Antigravity, Hermes).
It ingests heterogeneous exports and live captures into a set of SQLite files,
derives read models, and serves them through a query-first CLI, an MCP server,
a Python API, and an HTTP daemon. Pure Python.

This file holds repository semantics. Tasks live in external Beads (`bd`
redirects outside the checkout); job, worktree, and publication mechanics are
environment-level.

## Public boundary

Tracked content, commits, CI logs, and PR and review text are public. Never
commit operator archives, transcripts, private exports, local databases,
receipts, or scratch state. Tests use neutral synthetic fixtures.

## Orientation

```text
sources/ ─detect→ pipeline/ ─hash+write→ storage/{6 tiers} ─materialize→ analysis/
                                              │                              │
                            surfaces: cli/  mcp/  api/  daemon/  ─read-through─┘
                            verification:   devtools/  tests/  schemas/
```

New semantics go into the substrate (`storage`, `analysis`) or the product
layer first; surfaces adapt through `analysis`, `operations`, or `api`.
Surface-to-substrate imports are a ratchet (`devtools gate layering`: the
baseline may shrink, never grow); substrate-to-surface imports are forbidden.

Before a storage, daemon, MCP, source, or query change, read its
`docs/atlas/` sheet and confirm the claims you rely on against current code.

## Identity and content

- Sessions, messages, and blocks are `STRICT` tables whose IDs are generated
  from source identity. A message uses its provider-native ID when present;
  otherwise a digest of its declared semantic fields plus an occurrence
  counter, never its position, so an export that gains a message cannot
  re-point a durable `user.db` reference. Formulas and the field partition are
  in `pipeline/ids.py` and `docs/atlas/storage.md`.
- `messages.material_origin` records who authored the content, independent of
  role. `blocks.tool_outcome` is the structural outcome; a deliberate
  `unknown` is never success.
- Lineage: forks, resumes, subagents, and compaction replay the parent's
  prefix. The writer stores only the child's divergent tail, its
  `branch_point_message_id` (deliberately not a foreign key, so a parent's
  full replace cannot null it), and the inheritance mode; reads recompose.
  `session_links` also holds parser-asserted topology edges, resolved on every
  save by `write_parsed_session_to_archive`, the one write path shared by live
  ingest and replay.
- Archive writes are idempotent by content hash (SHA-256 over the canonical
  JSON payload, excluding user metadata, so tagging never re-imports). Only
  declared prose fields are NFC-folded; identifiers, tool arguments, paths,
  metadata, and mapping keys hash exactly, and absence stays JSON `null`,
  distinct from `""`. `pipeline/ids.py` declares which parser fields are
  hashed, which are prose, and why each excluded field is excluded.

## Storage tiers

| Tier            | Durability                  | Holds                                                                        |
| --------------- | --------------------------- | ---------------------------------------------------------------------------- |
| `source.db`     | durable                     | raw acquired bytes, artifact taxonomy, blob and GC substrate, hooks, sidecars |
| `index.db`      | rebuildable                 | parsed tree, FTS, links, costs, materialized insights                         |
| `embeddings.db` | expensive to rebuild        | vectors and their status; repurchased, never replayed, so no product route replaces this tier |
| `user.db`       | durable, irreplaceable      | assertions, settings, annotation schemas and provenance                       |
| `audit.db`      | durable, continuity-chained | previews, authorizations, attempts, continuity                                |
| `ops.db`        | disposable                  | cursors, attempts, convergence debt, daemon telemetry                         |

- Rebuildable state is never the authority for a durable mutation.
- A mutable SQLite source is retained as the canonical logical export of its
  declared `logical_tables` (`sources/sqlite_export.py`); that export's blob
  hash is its revision. A page image cannot be proven against a live database.
- The fresh archive format starts every tier at `PRAGMA user_version=1`.
  Earlier Polylogue state is moved aside intact as salvage evidence only: no
  migration, import, readback, or rollback. Explicitly declared external
  source files may be ingested into the empty archive.
- Later durable-tier changes (`source`, `user`, `audit`) are additive numbered
  migrations under `storage/sqlite/migrations/`, one step at a time. Derived
  tiers have no migration chain. For `index` and `ops`,
  `archive_tiers/schema_identity.py` stamps an identity over their DDL plus
  the lowering, materializer, and replay-routing fingerprints, every open
  compares it, and a mismatch is a typed `SchemaSkew` resolved by
  reconvergence through the daemon. `embeddings` carries no schema identity;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sinity/polylogue](https://github.com/Sinity/polylogue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
