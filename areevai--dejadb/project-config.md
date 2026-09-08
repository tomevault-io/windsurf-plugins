---
trigger: always_on
description: Embedded memory engine for AI agents — reference implementation of OMS (Open
---

# DejaDB

Embedded memory engine for AI agents — reference implementation of OMS (Open
Memory Spec). Rust workspace of 12 crates (plus `dejadb-js`, a standalone napi
package built outside the workspace). Memories are immutable
content-addressed grains in per-file Turso databases, queried with CAL, and
rendered into model-ready context in-process (no server in the recall path).

**Status**: published — the library crates + the `deja` binary on crates.io,
`dejadb` on PyPI and npm (`dejadb-py` and `dejadb-bench` stay `publish = false`).
The version lives in `[workspace.package]` in the root `Cargo.toml` (all crates
inherit it) and `CHANGELOG.md` records each release — don't restate the number
here (it goes stale). `ARCHITECTURE.md` is the design source of truth — the
architecture and the numbered design decisions; `CHANGELOG.md` summarizes what
exists; `crates/dejadb-bench/RESULTS.md` has the benchmark numbers.

## Commands

```bash
cargo test --workspace            # full suite (~950 tests, fast)
cargo test -p dejadb-cal          # per-crate
cargo run --release -p dejadb-store --example bench       # latency gates
cargo run --release -p dejadb-store --example voice_loop  # 50ms-cadence gate
cargo run -p dejadb -- recall --db demo.db --ns caller --subject john
```

- **Do not run blanket `cargo fmt`** — the tree is not uniformly rustfmt-clean
  (~177 files differ). Match surrounding style; format only
  the lines you touch.
- If CLI/MCP smoke tests fail with "spawn dejadb: No such file or directory":
  the cached test binary has a stale absolute path baked in via
  `CARGO_BIN_EXE_dejadb` (happens after the repo folder moves/renames).
  Fix: `touch crates/dejadb-cli/tests/*.rs` and re-run.
- CI (`.github/workflows/ci.yml`): test on ubuntu/macos/windows, clippy
  (`-D warnings`), MSRV build, `cargo doc`, coverage, Python (maturin + pytest),
  and Node (napi build + `node --test`). `security.yml` runs `cargo deny`.
  Still run tests locally before pushing.

## Workspace (dependency order)

```
memory stack:  dejadb-core ← dejadb-store ← dejadb-cal ← dejadb-context ┐
loop engine: deja-loop ← dejadb-loop (adapter) · dejadb-llm (providers) ┤
                              both feed the leaf crates ↓                 ↓
             dejadb-mcp, dejadb-server, dejadb-py, dejadb (binary), dejadb-bench
```

| Crate | What | CLAUDE.md |
|---|---|---|
| `dejadb-core` | `.mg` format, canonical serialization, content addressing, 12 grain types, tool-schema rendering | yes |
| `dejadb-store` | The store: dictionary-encoded triples, hybrid recall, heads/forks, bundles, CAS blobs (encrypted under an HKDF-derived subkey when the memory is), DSAR `subject_report`, declarative `retention:<ns>` policies, memory-tool adapter, migration importers. Backend-agnostic logic over an internal `Db` seam — embedded Turso (default) or PostgreSQL (`feature = "postgres"`, one memory = one schema, advisory-locked single writer, pgvector) | yes |
| `dejadb-conformance` | Backend-parameterized conformance suite (`publish = false`) — one case list (forks, replication, tombstones, PITR, BM25, vectors, CAS, CAL smoke) run against BOTH backends; the Pg runner needs `DATABASE_URL`/`DEJADB_PG_URL` and hard-fails when `CI=true` without one | — |
| `dejadb-cal` | CAL lexer/parser/executor, ASSEMBLE, `DejaDbFacade` + mounts | yes |
| `dejadb-context` | Budget-aware SML/TOON/Markdown/JSON rendering | yes |
| `deja-loop` | Substrate-agnostic self-improvement engine: `OmsSubstrate`/`LlmBackend` traits, 12 analyzers (incl. default-off `retention_sweep`), four gates, recommendation lifecycle, LLM DISCOVER→GROUND→VERIFY verifier, outcome measurement (no DejaDB deps) — `docs/loop.md` | — |
| `dejadb-loop` | DejaDB substrate adapter for Deja Loop (`deja_loop::OmsSubstrate` over `DejaDbFacade`) + recall-telemetry sidecar | — |
| `dejadb-llm` | Out-of-box LLM backends (OpenAI-compatible/Anthropic/Ollama over a small blocking HTTP client) for Deja Loop + the `remember()` free-text→Fact extraction (`extract.rs`) | — |
| `dejadb-mcp` | Stdio MCP server (see below) | — |
| `dejadb-server` | Web console + dejad hub (see below) | — |
| `dejadb` | The `deja` binary (see below) | — |
| `dejadb-py` | PyO3 bindings (see below) | — |
| `dejadb-bench` | Reproducible benchmark harnesses (latency, honesty, LoCoMo accuracy) | — |
| `dejadb-js` | Node (napi) bindings — **standalone package, not a workspace member** (see below) | — |

## Cross-cutting invariants

1. **Grains are immutable and content-addressed** (SHA-256 over the whole
   `.mg` blob). Nothing ever edits a stored blob; every edit is a
   supersession, every removal a tombstone (`forget`) or crypto-erasure.
   Store code mutates the *index layer* only.
2. **Canonical serialization is frozen** (NFC, sorted keys, compact keys,
   omit-defaults). Changing it silently changes every content address and
   breaks OMS conformance — see `crates/dejadb-core/CLAUDE.md`.
3. **CAL destruction is authorization-gated, not structural** (CAL 1.3,
   [`docs/cal-all-you-need-proposal.md`](docs/cal-all-you-need-proposal.md)).
   The destructive statements are `FORGET <hash>` (single-grain tombstone,
   `delete` verb), `FORGET SUBJECT "<id>" [WITH text_mentions]` (identity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AreevAI/dejadb](https://github.com/AreevAI/dejadb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
