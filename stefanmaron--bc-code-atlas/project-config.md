---
trigger: always_on
description: Build and operate a public, MCP-queryable code+docs index of Microsoft
---

# CLAUDE.md — bc-code-atlas

## Mission

Build and operate a public, MCP-queryable code+docs index of Microsoft
Dynamics 365 Business Central (AL language) — semantic search over source
and docs, plus an exact structural relationship graph (`calls`,
`subscribes`, `extends`) — reachable by any BC/AL developer's coding agent,
across every country localization and every shipped version, not just one.

This project has graduated past its original single-version local
proof-of-concept (see `REPORT.md` for that phase's findings — setup
friction, indexing benchmarks, real test-scenario transcripts, and a go
recommendation) into building the real thing. **The project constitution at
`.specify/memory/constitution.md` is now the authoritative source for
architectural principles** — read it first. This file is operational
context and history: what's been learned, what's already built, and
pointers to where the durable rules live. Where anything below conflicts
with the constitution, the constitution wins.

Work on new features goes through spec-kit
(`/speckit-specify` → `/speckit-plan` → `/speckit-tasks` →
`/speckit-implement`), not by appending ad hoc instructions to this file.

You are a fresh agent with no memory of the conversations that produced
this brief. Ask the user (via chat, not by guessing) if something here is
ambiguous enough to block a real decision — otherwise use judgment and
proceed.

## Where things stand today

**Built and running** — the multi-country/multi-version serving layer
(spec `specs/001-multi-version-serving/`) is implemented and verified live,
end to end, through the aggregator:
- `w1-28` AL source + two docs corpora (`dynamics365smb-docs`
  business-central docs, `dynamics365smb-devitpro-pb` AL developer/compiler
  reference) indexed via a custom `tree-sitter-al` chunker into
  `cocoindex-code`, served as MCP over HTTP (`chunker/`) — still the
  always-warm default corpus every search/graph tool falls back to.
- The structural graph extracted via the `graphify-al` fork, served as MCP
  over HTTP (`tools/graphify-al`), including on-demand exact-source lookup
  tools (`bcatlas_get_signature`, `bcatlas_get_procedure_body`,
  `bcatlas_get_object_source`) that re-parse real source on request rather
  than caching text in the graph.
- **Registry** (`registry/`, `:8803`) — `bcatlas_list_countries`,
  `bcatlas_list_versions`, `bcatlas_resolve_version` (real git ls-remote/log
  against the upstream repo, no new database); `bcatlas_diff` (file- or
  symbol-scoped, rejects unscoped requests) and `bcatlas_symbol_history`
  (multi-step change chain, filters out commits that touched a symbol's
  file without changing the symbol's own text).
- **Build** (`build/`, `:8804`) — `bcatlas_request_version` /
  `bcatlas_version_status`: staging + atomic promote, bounded GPU-aware
  build queue with request coalescing, clone-and-patch incremental builds
  against the nearest already-warm sibling (reuses cocoindex-code's stock
  incremental `ccc index`, no fork), LRU/TTL eviction of idle warm data.
- `chunker/mcp_http_server.py` and `tools/graphify-al/graphify/serve.py`
  are now multi-tenant: every search/graph tool accepts optional
  `country`/`version` (the exact `commit_sha`, not `version_string`) to
  route to a specific built pair instead of the default corpus.
- A thin aggregator (`aggregator/`) presenting one `/mcp` endpoint to
  clients, forwarding to all four backends and passing `country`/`version`
  through unchanged.
- Real tester validation against both original test scenarios plus organic
  use; see `REPORT.md` for the full account and `README.md` for the
  current architecture diagram and local Quick Start.

**Verified live this build** (not simulated — a real MCP client session
against the running aggregator): version discovery/resolution against the
real upstream repo (including two real bugs found and fixed along the way
— transitive-ancestor major/minor leakage, and `-vNext` preview builds
outranking stable ones under naive "highest build wins"); an unscoped diff
rejected explicitly; a real symbol diff and a real multi-step symbol
history that correctly collapsed 2 raw touching commits down to 1 real
change; a genuinely new (country, version) build requested, queued, and
built for real through the actual build queue (not an ad hoc script),
confirmed via `bcatlas_version_status` and the promoted artifact on disk.

**Known open items, not yet fully closed:**
- ~~No trustworthy incremental-vs-cold wall-clock number has been
  captured yet~~ — now measured for real, twice, on both sides (see "Key
  facts already established" below): builds are fast (minutes), the
  shared serving daemon's cold reindex is genuinely slow (many hours on
  the hosted VM's 4-vCPU hardware), not the ~30min/~2hr originally
  assumed. This asymmetry is expected and fine — see that section for why.
- The hosted search daemon's stall-recovery watchdog killed the *wrong*
  process for an unknown amount of past time (fixed, not yet re-verified
  end to end): `daemon.pid` can go stale mid-run, pointing at an
  already-dead sibling from an earlier spawn attempt while the real,
  actively-computing daemon runs on as a completely untracked process —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StefanMaron/bc-code-atlas](https://github.com/StefanMaron/bc-code-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
