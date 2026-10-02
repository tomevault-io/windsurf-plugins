---
trigger: always_on
description: A curated, local corpus of Ethereum research and standards with hybrid search,
---

# CLAUDE.md

## What this is

A curated, local corpus of Ethereum research and standards with hybrid search,
exposed to LLM clients over MCP. Sources are declared in `sources.toml` and
fetched by pluggable adapters (`discourse`, `repo`, `feed`).

Two uses, both first-class: **research recall** over nine years of
ethresear.ch and EthMagicians (what was argued, by whom, and when it was
superseded), and **spec engineering** against the EIPs, ERCs, and
consensus-specs (what a constant's value is, what a spec function does, per
fork). The second is why `lookup_spec` exists — ranked search alone kept
burying spec documents under forum volume.

**Out of scope for now:** the public web frontend. No UI, no user accounts, no
request handlers of our own. If a task seems to need them, stop and ask.

The MCP server's HTTP transport (`wikipethia mcp --http`) is not an exception to
that: it mounts rmcp's own tower service and adds no handlers, auth, or
pages. It has **no authentication**, and the bare port must bind loopback or a
private interface — never the internet directly. Public exposure is sanctioned
in exactly one shape (decided for M15): a TLS reverse proxy with rate limiting
in front of the loopback bind, serving read-only public data to any MCP
client. That deployment lives in `deploy/`; auth, health endpoints, and
anything needing a handler of our own remain out of scope.

## Stack

- Rust, 2024 edition, cargo workspace
- SQLite (WAL) + FTS5 + sqlite-vec — one file, no daemon
- `rmcp` for the MCP server
- Embeddings behind a trait; default impl is local via `fastembed`

## Layout

```
wikipethia-core/    documents, parsing, chunking, spec extraction, index, search
                — no I/O beyond the DB
wikipethia-embed/   the fastembed Embedder impl — model cache and its one-time download
wikipethia-fetch/   HTTP client, rate limiting, adapters — all crawl network lives here
wikipethia-mcp/     MCP server library — the `wikipethia mcp` subcommand
                (stdio by default, streamable HTTP with --http); builds no
                binary of its own
wikipethia/         the one binary: sync, index, embed, update, search, status,
                dedup, eval, agent-eval, publish, mcp
sources.toml        the manifest — source of truth for what is in the corpus
deploy/             the hosted-endpoint shape: systemd units, nginx config,
                runbook — config only, no crate; cargo and CI never look at it
```

## Commands

```
cargo build --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

In this repo, `cargo run -p wikipethia -- <cmd>`; installed, just
`wikipethia <cmd>`. Every crate, directory, and binary is named `wikipethia*`
— renamed from `corpus-*` on 2026-08-21, because `cargo install --path
corpus-cli` for a tool called wikipethia was confusing at exactly the moment
a new user meets it.

```
wikipethia build  [--source <id>]      # clone day: sync + index + embed
wikipethia update [--source <id>]      # same three stages, incrementally
wikipethia status                      # docs/vectors per source; READY or not
wikipethia sync   [--source <id>] [--limit N] [--full] [--force] [--db <path>]
wikipethia index  [--source <id>] [--force]
wikipethia embed  [--force]
wikipethia search "<query>" [--limit N]
wikipethia dedup  [--threshold 0.95] [--source <id>]
wikipethia eval                        # retrieval: recall@10
wikipethia agent-eval [--limit N] [--model haiku]
wikipethia agent-eval --regrade <dir>  # re-score, no spend
wikipethia publish [--tag <t>] [--out dist] [--dry-run]  # maintainer: snapshot → zstd → GitHub release
wikipethia mcp [--db <path>] [--http <addr> [--allow-host <name>] [--public-bind]]
```

`build` and `update` are the same pipeline; they differ in what they report,
and `update` is the one meant for a timer. `refresh` is a kept alias for
`update`. Every stage is incremental, so either is safe to run at any time.

`sync --full` widens an incremental walk to every page; `sync --force`
refetches regardless of what is on disk. Together they are the only way to
see a post edited in place — that moves no upstream timestamp — and they
cost what the first crawl cost. Not a routine.

Sync checkpoints live in the database (`meta`, `checkpoint.<id>`), so a
published corpus carries them and a downloader's `update` walks
incrementally. `publish` also stamps `mirror.absent.<id>` into the snapshot:
while set, sync declines the full-listings walk and index skips its prune
pass (both assume the local raw mirror is complete, and a download ships
without one). Cleared only on real evidence of a rebuilt mirror — a
completed `sync --full` for a forum, a checkpoint-advancing tarball run for
a repo, never for a feed (a feed can only re-mirror what its live feed.xml
still lists).

`index` and `embed` take an advisory lock on the database (a `meta` row).
A second writer fails fast rather than interleaving: `chunks.id` has no
`AUTOINCREMENT`, so rowid reuse can otherwise attach a vector to text it was
not computed from, silently. Readers, including `wikipethia mcp`, never take it.

`agent-eval` spawns a headless Claude Code session per question and consumes
real usage (API credit or plan allowance, depending on how the `claude` CLI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JossDuff/wikipethia](https://github.com/JossDuff/wikipethia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
