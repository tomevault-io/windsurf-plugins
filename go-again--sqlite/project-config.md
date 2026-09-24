---
trigger: always_on
description: Onboarding for AI agents and humans **developing** this repository. Read it top-to-bottom to be useful within ~5 minutes. (Agents *using* the package as a dependency want [`skills/`](skills/) instead; end users want [`README.md`](README.md) and [`docs/`](docs/).)
---

# AGENTS.md

Onboarding for AI agents and humans **developing** this repository. Read it top-to-bottom to be useful within ~5 minutes. (Agents *using* the package as a dependency want [`skills/`](skills/) instead; end users want [`README.md`](README.md) and [`docs/`](docs/).)

This file is canonical; `CLAUDE.md` is a pointer to it. Deep C-ABI internals live in [`dev/architecture.md`](dev/architecture.md).

---

## What this package is

`gosqlite.org` is a **CGo-free** SQLite driver for Go, a drop-in replacement for:

- `github.com/mattn/go-sqlite3` — the C-bound driver; we register as `"sqlite3"`.
- `modernc.org/sqlite` — the upstream CGo-free driver this fork builds on; we register as `"sqlite"`.
- `github.com/glebarez/sqlite` and `gorm.io/driver/sqlite` — gorm dialectors; ours is the `gorm/` sub-package.

Plus first-class typed Go APIs for **sqlite-vec** vector search and **FTS5** full-text search, encryption-at-rest, a user-implementable VFS, a bounded page cache, and a catalog of loadable Go SQL extensions.

Supported Go: the **two most recent releases** (the pin lives in `go.mod`; don't name versions in prose). Modern syntax is a feature — generics, `iter.Seq2`, `log/slog`, generic type aliases, range-over-int, `sync.WaitGroup.Go`, `strings.SplitSeq`, `reflect.TypeFor` are all in use. `just lint` runs `gopls modernize` to enforce the policy.

## Architecture in one paragraph

We **fork modernc's hand-written Go wrapper** (the root `*.go` files) so we can add per-conn methods and own the `Driver` type; the transpiled C (`lib/`, `vec/`, `vfs/c/`) stays an untouched external dependency. The wrapper talks to that C through a `uintptr`/`unsafe.Pointer` function-pointer dance centralized in `internal/cabi`. The weird-looking patterns (uintptr arithmetic, empty mutex critical sections, named-field struct literals) are the contract, not bugs — **don't restyle them**. Full rationale, the cabi primitives, and the struct-drift discipline: [`dev/architecture.md`](dev/architecture.md).

---

## Repository layout

The high-signal map is the root-package fork surface; each sub-package documents its contract in a `doc.go`.

**Root (modernc-derived + our additions):**

```
sqlite.go               init(), driver registration ("sqlite" + "sqlite3")
driver.go               *Driver with Extensions / ConnectHook; dispatches remote-scheme DSNs
remote.go               RegisterRemoteScheme — teaches the "sqlite" driver to open network DSNs (sql.Open("sqlite","quicsql://…")); the quicSQL seam, no network dep in the root
conn.go                 *conn (alias *Conn); most of the work happens here
stmt.go, rows.go, result.go, tx.go
error.go                *Error with Code() / ExtendedCode()
convert.go              SQLite ↔ Go value coercion
backup.go / backup_factory.go   *Backup + (*Conn).Backup + Serialize/Deserialize
blob.go                 *Blob + (*Conn).OpenBlob — incremental BLOB I/O
extension.go            LoadExtension / EnableLoadExtension
limits.go               GetLimit / SetLimit
hooks.go                Update / Authorizer / Trace hooks
rtree.go                (*Conn).RegisterRTreeGeometry / RegisterRTreeQuery
session.go              SESSION ext — CreateSession/ApplyChangeset/Invert/Concat + *Session
pre_update_hook.go      RegisterPreUpdateHook / Commit / Rollback
fcntl.go                file-control helpers (incl. EnableChecksums)
wal.go                  WALCheckpoint / WALAutoCheckpoint / RegisterWALHook
snapshot.go             GetSnapshot / OpenSnapshot / SnapshotRecover + *Snapshot
control.go              SetProgressHandler / SetDBConfig / QueryDBConfig
vtab.go / module.go     virtual-table trampolines + CreateModule / CreateModuleSplit
pointer.go              sqlite.Pointer for binding Go values into SQL params
mutex.go                unlock_notify mutex wrapper
dsn.go                  mattn `_*` DSN-flag translator
constants.go            SQLITE_* re-exports + ErrNo / ErrNoExtended
compat_*.go             type aliases + reflective RegisterFunc / RegisterAggregator + conversion
stmt_cache.go           per-conn prepared-stmt LRU + StmtCacheStats
introspect.go           TableColumnMetadata / Status / TxnState + (*Stmt).Readonly / Status
config.go / open_config.go   sqlite.Config / Pragmas / Encryption / TxLock + sqlite.Open(Config)
doc.go                  package doc (pkg.go.dev landing)
```

**Sub-packages** (each has a `doc.go`):

- `gorm/` — gorm dialector, a **separate module** (`gosqlite.org/gorm`, its own `go.mod`); the core module does not depend on `gorm.io/gorm`. Originally from glebarez; diverged: RETURNING always-on, OpenConfig, error translator. `DropTableHook` (a public extension point for third-party plugins) is defined here. Examples under `gorm/examples/`.
- `vec/` — sqlite-vec typed `Table`.
- `fts/` — FTS5 typed `Index[K, V]`.
- `fusion/` — RRF / RRF2 rank-fusion helpers (pure Go, no SQLite dep).
- `sqlitex/` — ergonomic `database/sql` helpers (Save, Transaction, ExecScript, Execute, Result*, Migrate) in the zombiezen/crawshaw lineage.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-again/sqlite](https://github.com/go-again/sqlite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
