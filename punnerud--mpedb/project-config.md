---
trigger: always_on
description: Embedded multi-process shared-memory database in Rust: sqlite's operational model
---

# mpedb

Embedded multi-process shared-memory database in Rust: sqlite's operational model
(no server, processes attach and may be SIGKILLed at any instant) + PostgreSQL-grade
concurrency (MVCC snapshots, lock-free readers) + rigid schema validation that sqlite
lacks. SQL compiles once to content-hashed plans (`execute(hash, params)` hot path with
zero parsing). **Read design/DESIGN.md before touching concurrency, lock, or commit-path code —
every protocol there survived a 37-finding adversarial review, and the ordering rules
(fences, meta publication, slot generation-CAS) are load-bearing.**

SQL is user-extensible via stored PySpell functions and `:sym:` operator
macros — SQL-EXTENSIONS.md is the contract; `mpedb fn list` / `mpedb op list`
show what a given database defines. The workload model (design/
DESIGN-MODEL-LANG.md, `mpedb model show`) declares what a database is FOR.

Measured comparisons all live in `benchmarks/` (index: `benchmarks/README.md`) —
head-to-head, OLAP/vector/graph, and the LISTEN/NOTIFY cell against PostgreSQL.
Three rules bind every one of them: a control arm for anything claimed to cost
something, like-for-like durability (#122 — never mpedb's weaker mode against a
log-based engine), and the hardware published when the hardware is the answer.

## Commands

- Build/test all: `cargo test --workspace` (mpedb-capi is its OWN workspace —
  it exports `sqlite3_*` and cannot co-link the bundled sqlite the parent's
  feature unification pulls in; test it with
  `cargo test --manifest-path crates/mpedb-capi/Cargo.toml`)
- One crate: `cargo test -p mpedb-core` (also: mpedb-types, mpedb-sql, mpedb, mpedb-cli)
- Lint (keep clean): `cargo clippy --workspace --all-targets -- -D warnings`
  **plus the three satellite workspaces**, which `--workspace` cannot reach —
  a separate workspace is invisible to the parent's `--workspace`, which is how
  a lint once reached main red with the documented command passing locally:

      for m in capi pg fs; do
        cargo clippy --manifest-path crates/mpedb-$m/Cargo.toml \
          --all-targets -- -D warnings
      done

  CI runs all three now (`capi` and `pg` on Linux and macOS, `fs` on Linux
  only), so the documented procedure and the enforced one match. The two
  platform limits are measured, not assumed: `mpedb-pg` uses `std::os::fd` and
  `std::os::unix` so it does not build for Windows, and `fuser`'s build script
  wants macFUSE through pkg-config so `mpedb-fs` does not build on Apple.
- Slow/instrumented tests are `#[ignore]`d: `cargo test -p mpedb-core -- --ignored`
- **Point every test's scratch at a real volume**: `MPEDB_TEST_DIR=/path cargo test …`.
  `mpedb_testkit::scratch_base` (and `mpedb_core::scratch_dir`, which spells the
  same knob because the testkit depends on the facade that depends on the core)
  read it. Without it the suite writes to the root filesystem, and a full disk
  does not announce itself as one: it has surfaced as a flapping measurement, as
  two corpus files "hanging", and as `ld: signal 7 [Bus error]` asking for an
  LLVM bug report.

## Crate map (dependency order)

- `crates/mpedb-types` — shared, dependency-light: Value/ColumnType, Schema + canonical
  bytes + blake3 hash, TOML Config, memcmp-ordered key encoding (`keycode`), stack-based
  expression IR with SQL 3VL (`expr`), plan Footprint/PlanHash. Everything decodable is
  bounds-checked: corrupt input must yield `Error::Corrupt`, never a panic.
- `crates/mpedb-core` — the engine. `backup` (RAW whole-database copy:
  `Database::backup()` / `restore_backup()`, `mpedb backup|restore` — the
  data pages up to the snapshot's high water, so the file is sized by the
  DATA and not by the arena. Takes NO lock: committed pages are immutable
  and a pinned reader holds the reuse floor, so the snapshot is frozen
  while writers continue. The lock/reader/ring pages are NOT copied — they
  are process state — which is why `max_readers` rides in the header and is
  checked: it decides where the data region starts, so a mismatch would
  place every page at the wrong id, silently), `pagestore` (COW page
  discipline; in-memory TestStore
  for model tests), `btree` (COW B+tree, overflow chains, model-tested against BTreeMap),
  `row` (null bitmap + fixed + varlen codec), `shm` (mmap, init via flock+fallocate, meta
  double-buffer with atomics/fences, robust ERRORCHECK mutex, reader table with packed
  {pid,seq} generation words + /proc start-time identity), `engine/` (split into
  mod/read/write/freelist/commit: ReadTxn/WriteTxn, catalog, chunked freelist with
  commit-time fixpoint, typed row API, page-accounting verifier).
- `crates/mpedb-sql` — tokenizer → AST → binder (rigid types, param unification, const
  folding) → `planner/` (select/join/aggregate/access/footprint: PkPoint/PkRange/
  IndexPoint/FullScan + footprints) → CompiledPlan in `plan/` (encode/decode/validate/
  explain: canonical bytes, blake3 hash, fully re-validating decode).
- `crates/mpedb` — facade: Database::open(config), prepare/execute/query, WriteSession,
  shared plan registry in the catalog's sys-keyspace (`plan/<hash>`), CHECK compilation,
  the plan executor in `exec/` (mod = TxnCtx + exec_stmt, gather, aggregate), and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [punnerud/mpedb](https://github.com/punnerud/mpedb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
