---
trigger: always_on
description: CodeGraph is a deterministic tree-sitter + SQLite/FTS5 code knowledge graph. The
---

# AGENTS.md — codegraph-rs contributor contract

CodeGraph is a deterministic tree-sitter + SQLite/FTS5 code knowledge graph. The
single `codegraph` binary indexes source, resolves relationships, answers graph
queries through the CLI and MCP, and can keep an index current through a local
daemon. It contains no AI, vector database, embedding model, or LLM runtime.

This file is the canonical instruction source for coding agents. `CLAUDE.md`
imports it; do not create a second agent guide. Detailed, changeable behavior
belongs in [`docs/`](docs/README.md), not here.

## Navigate with CodeGraph first

Before researching source in an indexed checkout, verify the index:

```bash
codegraph status . --json
```

Lifecycle commands take an optional positional project path. Research commands
take one query/target plus `-p/--path`:

```bash
codegraph status . --json
codegraph sync .
codegraph explore "index and resolve flow" -p .
codegraph search "ReferenceResolver" -p .
codegraph node "ReferenceResolver" -p .
```

Use `explore` for an area or call flow, `search` for a known name, `node` for one
symbol/file plus its trail, and `impact` before a refactor. Do not append `.` as
a second positional argument to a research command. If the index is unavailable,
follow `status` recovery guidance; do not initialize or rebuild unless requested
or required by that guidance.

## Hard invariants

1. **Deterministic graph output.** Identical source and configuration must produce
   identical canonical nodes, edges, references, files, and query ordering.
   Incremental `sync` must converge with a clean full index.
2. **Golden compatibility.** Extraction goldens under `reference/golden/` are
   byte-stable canonical artifacts. Update them only for an intentional graph
   behavior change, with the matching corpus and regeneration evidence documented
   in [`docs/equivalence.md`](docs/equivalence.md).
3. **Stable node IDs.** Symbol IDs are
   `{kind}:{sha256("{filePath}:{kind}:{name}:{line}").hex[..32]}`; file nodes are
   `file:{relative/path}`. Paths use `/`; lines are 1-based. Within one file's
   extraction, a later declaration that collides with an earlier one at a
   different column appends `:{column}` (zero-based UTF-16 code units, upstream
   #1349), so it no longer overwrites the first. Never change this formula
   incidentally.
4. **No AI/vector runtime.** Do not add AI, LLM, embedding, vector-database, or
   inference dependencies. `scripts/guardrail.sh` enforces the dependency boundary.
5. **Project containment.** Managed state stays under the selected project index
   root. Filesystem fallbacks must prove lexical containment before probing or
   reading a path. Do not broaden project authority through ancestor discovery,
   symlink guessing, environment-global state, or another project's configuration.
6. **Fail closed.** Ambiguous resolution stays unresolved. Unsafe stale source is
   served whole or omitted, never sliced using stale line ranges. Lock, checksum,
   migration, and release validation failures must stop rather than silently skip.
7. **Protocol and stdout purity.** MCP/JSON-RPC output owns stdout. Logs and
   diagnostics go to stderr. Additive fields are preferred; existing JSON, text,
   installer, and protocol contracts require explicit compatibility tests.
8. **No manual releases or version edits.** Release Please owns versions and tags.
   Distribution is GitHub Releases plus `cargo install --git`; no crate is
   published to crates.io.

## Workspace ownership

The workspace members are declared in root `Cargo.toml`; that manifest is the
authority when the list changes.

| Crate                            | Owns                                                                  |
| -------------------------------- | --------------------------------------------------------------------- |
| `codegraph-core`                 | shared types, config, IDs, file classification, logging               |
| `codegraph-extract`              | language detection, tree-sitter/custom extraction, scan policy        |
| `codegraph-store`                | SQLite schema/migrations, FTS5, persistence and queries               |
| `codegraph-resolve`              | import/name resolution and framework resolvers                        |
| `codegraph-graph`                | traversal, impact, search scoring and query parsing                   |
| `codegraph-mcp`                  | MCP schemas, rmcp transports, project resolution, tool rendering      |
| `codegraph-watch`                | incremental synchronization and filesystem watching                   |
| `codegraph-daemon`               | shared process lifecycle, IPC, registries and detach behavior         |
| `codegraph-ui`                   | browser viewer: loopback JSON API, live channel, embedded `ui/` build |
| `codegraph-cli` (`codegraph-rs`) | CLI, installer, orchestration and shipped binary                      |
| `codegraph-bench`                | equivalence oracle and reproducible benchmark harness; not shipped    |

Keep dependency direction acyclic and lower layers independent of presentation.
Extraction must not depend on store/graph. Query rendering must not leak into core

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sunerpy/codegraph-rust](https://github.com/sunerpy/codegraph-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
