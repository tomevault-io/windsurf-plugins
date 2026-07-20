---
trigger: always_on
description: Before making any changes, read and follow **[CONTRIBUTING.md](CONTRIBUTING.md)**.
---

# cached — AI Agent Instructions

## Contributing Guidelines
Before making any changes, read and follow **[CONTRIBUTING.md](CONTRIBUTING.md)**.
Key points:
- Run `make fmt` before committing
- Update `CHANGELOG.md` with a description of what changed and why
- After editing `src/lib.rs`, sync the README with `make docs` and verify with `make check/readme`
- After adding or removing Makefile targets, update `make help` and verify with `make check/help`
- Run `make ci` to validate the full pipeline before submitting

## Git Push Protocol
Before every `git push`, show a diff summary so the user can see exactly what is going up:

```bash
git log origin/BRANCH..HEAD --oneline   # commits being pushed
git diff origin/BRANCH --stat           # files changed
```

Follow with a one-sentence summary (e.g. "Pushing 2 commits touching src/lib.rs and CHANGELOG.md"). Then push.

---

## Temp Files
Write any scratch files, research dumps, or intermediate agent outputs to `local/` — it is gitignored and always safe to write to. Do not create temp files elsewhere in the repo.

---

## Project Overview
`cached` is a Rust crate providing generic cache implementations and simplified function memoization. Workspace members:
- `cached` — main crate (`src/`)
- `cached_proc_macro` — procedural macro crate (`cached_proc_macro/src/`)
- `cached_proc_macro_types` — shared types used by the proc macro (`cached_proc_macro_types/src/lib.rs`); currently just `Return<T>`
- `examples/wasm` — WASM example (separate Cargo workspace member)

---

## Toolchain & Edition
- **Rust edition: 2024.** MSRV is **1.89** (raised from the edition-2024 floor of 1.85 by redb 4.x), declared via `rust-version` in `Cargo.toml`, `cached_proc_macro/Cargo.toml`, and `cached_proc_macro_types/Cargo.toml`.
- **`rust-toolchain.toml` pins the toolchain to `1.96.0`** (latest stable) for local development and CI. Always build/format/test with this toolchain — it is what keeps `cargo fmt` deterministic.
- **If `cargo fmt --check` or `make ci` reports formatting diffs in files you did not touch, you are on the wrong rustfmt** — confirm `cargo --version` shows `1.96.0` (run `rustup toolchain install 1.96.0` if missing) instead of reformatting the whole tree.
- The pin is dev/CI-only: it is not published and does not affect downstream consumers, who only need Rust ≥ 1.89.

---

## Store Types (current names as of v3.0)

| Type | Module | Description |
|---|---|---|
| `UnboundCache<K,V>` | `cached::stores` | Unbounded HashMap-backed cache |
| `LruCache<K,V>` | `cached::stores` | LRU eviction, size-bounded |
| `TtlCache<K,V>` | `cached::stores` | Global TTL, no size limit; requires `time_stores` |
| `LruTtlCache<K,V>` | `cached::stores` | LRU + global TTL, size-bounded; requires `time_stores` |
| `TtlSortedCache<K,V>` | `cached::stores` | TTL-ordered, optional size limit; requires `time_stores` |
| `ExpiringLruCache<K,V>` | `cached::stores` | LRU, size-bounded, per-value expiry via `Expires` trait |
| `ExpiringCache<K,V>` | `cached::stores` | Unbounded HashMap-backed, per-value expiry via `Expires` trait; default store for `#[cached(expires = true)]` |
| `ShardedUnboundCache<K,V>` | `cached::stores` | Fully concurrent, sharded `Arc`-backed unbounded cache; default for `#[concurrent_cached]` (no extra attrs) |
| `ShardedLruCache<K,V>` | `cached::stores` | Fully concurrent, sharded LRU; default for `#[concurrent_cached(max_size = N)]` |
| `ShardedTtlCache<K,V>` | `cached::stores` | Fully concurrent, sharded TTL cache; default for `#[concurrent_cached(ttl_secs = N)]` (also selected by `ttl_millis` and `ttl = "<expr>"`); requires `time_stores` |
| `ShardedLruTtlCache<K,V>` | `cached::stores` | Fully concurrent, sharded LRU + TTL; default for `#[concurrent_cached(max_size = N, ttl_secs = N)]` (also selected by `ttl_millis` and `ttl = "<expr>"`); requires `time_stores` |
| `ShardedExpiringCache<K,V>` | `cached::stores` | Fully concurrent, sharded per-value expiry (unbounded); default for `#[concurrent_cached(expires = true)]` |
| `ShardedExpiringLruCache<K,V>` | `cached::stores` | Fully concurrent, sharded LRU + per-value expiry; default for `#[concurrent_cached(expires = true, max_size = N)]` |

**`ExpiringCache` / `ExpiringLruCache` notes:**
- Neither store requires the `time_stores` feature — they are always available.
- Neither `ExpiringCache` nor `ExpiringLruCache` exposes a public `store()` accessor; use the `Cached` trait API for inspection.
- `cache_get` and `cache_get_mut` on `ExpiringCache` use two hash lookups on the hit path due to stable Rust's NLL borrow-checker limitation: the first lookup checks expiry (dropping the reference via `.map`), the second returns the value reference or calls `remove_entry`. Polonius (nightly) would allow a single lookup. This is intentional and documented in the source with a comment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaemk/cached](https://github.com/jaemk/cached) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
