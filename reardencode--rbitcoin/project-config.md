---
trigger: always_on
description: Composition (has-a) is typically more clear and less error prone than
---

# Agent notes

## Prefer composition over inheritance in data models

Composition (has-a) is typically more clear and less error prone than
inheritance (is-a), although rust traits can make that blurry. Avoid tall
inheritance trees.

## Prefer immutable data structures

Immutable data structures built once then composed with other immutable
structures typically perform better than data structures mutated over time.

For example, prefer an immutable map that is built from the streamed results
of some work over a mutable hashmap. Even if the data structure itself is
mutable, in rust not having to make it mutable makes life better.

If we needed to add additional data to the members of a hashmap, we could
create an outer map that contains the additional info and annotates the
members of the inner map on read, converting them to the data type with the
additional fields (which will also have the inner object).

In short, prefer composition over mutation as well as composition over
inheritance.

## Store concurrency: lock-free by default

**Default: no locks on the store hot path.** Concurrency is **roles + publish
order + HWM**, not map mutexes (maps removed — phase 6).

| Rule | Detail |
|------|--------|
| Roles | At most **one Class A appender** and **one spend annotator** per process; **N readers** of published ranges always free |
| Publish | body → idx → count/HWM (Release); then head / `header_txs` as visibility requires |
| Capacity grow | fallocate/`set_len` only (no map epochs); readers use published HWM |
| Layout grow (`tx.head`) | **segment roll**: seal open head (fuse8) + create new fixed 25-bit head — no mono-file bits-widen |
| Class C tip | L2 write-behind; `flush_class_c_tip` **before** body-queue dequeue |
| Not OK | Long-held store locks on IBD/read path, “pause all queries during confirm”, multi appenders |

If a change introduces a new long-held store lock on the IBD/read path, it is the
wrong design — fix the protocol. See `docs/concurrency.md`.

## io_uring: do not flatten custom machines

**Under no circumstances** replace a purpose-built / multi-stage **io_uring
machine** (fused resolve, spend-annotate RMW, pipeline stages, depth-round
machines, etc.) with “simple” batched `pread`/`pwrite` / one-shot
`pread_batch`/`pwrite_batch` submission **without explicit permission from the
user**.

| OK | Not OK without permission |
|----|---------------------------|
| Fix bugs inside the existing machine | Delete/retire a custom machine and call bulk batch helpers instead |
| Thread new flags (e.g. DONTCACHE) through the same SQE path | “Simplify” to serial pread + one big submit for a path that had a staged machine |
| Fall back to pread when uring is unavailable (existing policy) | Rewrite a machine away “because batch is enough” |

If a change seems to require collapsing a machine, **stop and ask** — do not
land the simplification as a drive-by cleanup.

## Create pins: pipeline-local only (no process FIFO)

| Rule | Detail |
|------|--------|
| Pin material | **Plan / batch only** — `batch_pin`, `BatchParents`, plan-local **sparse** `external_parent_outs` (`SparseExternalPin`). SharedParentPin = immutable body compose. No process create pin FIFO |
| IBD confirm intake | **body queue wire only** → lookup → load (no hash-only / Class-A-only confirm) |
| Ancient parents | Cold Class A denserels into plan-local / BatchParents only |
| Header plans | **ConfirmParentCache** always on (MTP / tip-ahead headers) |
| Removed | **CreateResidency**, **OutFifo**, **archive sticky**, half-row / out-slim, **`RBITCOIN_CONFIRM_CACHE`**, **`RBITCOIN_RESIDENCY_BYTES`** |
| IBD sizes | **`conf_plans=`** + body-queue / pipeline meters (no `residency creates=`) |


## GitHub CI must stay green (every commit)

CI is [`.github/workflows/ci.yml`](.github/workflows/ci.yml) (push/PR to
`master`/`main`). **Do not push or leave a commit that would fail the required
`test` job.** A red CI on `master` is incomplete work.

### Required before each code commit (`test` job)

From `nix-shell` (or the same **rustc 1.82** class CI pins):

```bash
cargo fmt --all -- --check          # if dirty: cargo fmt --all
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

| Gate | Expectation |
|------|-------------|
| `cargo fmt --all -- --check` | Clean |
| `clippy … -D warnings` | Clean under `[workspace.lints.clippy]` allows in root `Cargo.toml` |
| `cargo test --workspace` | All non-ignored tests pass |

**Toolchain:** CI pins **rustc 1.82.0** (same class as `nix-shell` / crane). Do not
rely on host “latest stable” alone. Expand clippy allows only for real noise
after a toolchain bump — prefer fixing the code.

### Coverage job

`./scripts/coverage.sh` enforces **100% first-party HTML uncovered-line** (see
`COVERAGE.md`). It runs as a separate CI job (slow). Prefer running it when
touching store/query/consensus hot paths. **Do not land new uncovered production
lines.** The coverage job may be `continue-on-error` while historical gaps are
closed — that is temporary; required `test` must still pass.

If a change cannot pass required gates, **do not commit it as done** — fix, split,
or get explicit user approval for a temporary exception (prefer none).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reardencode/rbitcoin](https://github.com/reardencode/rbitcoin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
