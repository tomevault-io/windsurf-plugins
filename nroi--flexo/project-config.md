---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Flexo is a caching proxy for `pacman`, the Arch Linux package manager. It sits between Arch clients
and the official mirrors: it picks low-latency mirrors automatically, caches downloaded packages, and
shares a single upstream connection across multiple clients downloading the same file simultaneously.

It is a single Rust binary that listens on TCP (default port 7878) and speaks HTTP by hand (no async
runtime, no web framework). Concurrency is one OS thread per client connection plus background
worker threads for downloads. Fetching uses `curl`; serving cached files uses `sendfile` via `libc`.

## Repository layout

The Cargo project is **not** at the repo root — it lives in the `flexo/` subdirectory. Always run
cargo commands from there.

```
flexo/            <- the Rust crate (run cargo here)
  src/
  conf/flexo.toml <- annotated example config
test/             <- docker-compose integration tests + helper crates (tcp-proxy-delay, integration-test-client)
.github/workflows/rust.yml
mirror_selection.md  <- design notes on how mirrors are scored/selected
```

## Build, test, run

All cargo commands run from `flexo/`:

```bash
cargo build
cargo test                              # unit + in-crate tests
cargo test test_filesize_exceeds_sendfile_count   # single test by name
cargo run                               # reads /etc/flexo/flexo.toml or FLEXO_* env vars
```

CI (`.github/workflows/rust.yml`) just runs `cargo build` and `cargo test` with working-directory
`./flexo` on push to `master`/`dev` and PRs to `master`.

### Integration tests (Docker)

End-to-end tests spin up mock mirrors (fast/slow/stalling/redirecting/no-content-length) plus Flexo
servers and a client, and assert behavior under adverse mirror conditions. Run from
`test/docker-test-local/`:

```bash
./docker-compose          # builds tarballs of the source, then runs docker-compose up
```

The script tars `flexo/src` + `Cargo.toml`/`Cargo.lock` into the test images, so the integration
tests always build from the current working-tree source. The run exits with the flexo-client's exit
code. Each `mirror-*-mock` and `flexo-server-*` directory under `docker-test-local/` is one scenario.

## Architecture

### Generic scheduling core (`src/lib.rs`)

`lib.rs` is a **provider-agnostic job-scheduling framework** expressed entirely through traits — it
contains no Arch/pacman/HTTP specifics. The key abstractions:

- `Job` — a unit of work with a large associated-type bundle (`Order`, `Provider`, `Channel`,
  score type `S`, properties `PR`, etc.).
- `Provider` — a source that can serve jobs (here: a mirror). Has an `initial_score` (latency-test
  derived, known before use) and accrues a dynamic score from real success/failure.
- `Order` — a request for a specific artifact; knows how to open/reuse a `Channel`.
- `Channel` — a reusable connection (enables persistent connections across downloads).
- `JobContext` — the orchestrator. `try_schedule()` is the entry point: it deduplicates in-progress
  orders, picks the best provider (`ProviderGuards` / `provider_guards.rs` manage per-provider
  locking and choice), reuses or opens channels, and retries on failure (`NUM_MAX_ATTEMPTS`,
  `TIMEOUT_ALL_RETRIES`). It returns a `ScheduleOutcome`: `Cached`, `Scheduled`, `AlreadyInProgress`,
  `Uncacheable`, or `Unavailable`.

When changing scheduling/retry/provider-selection logic, this is the file — but note that a lot of
the retry policy lives here while the per-request HTTP response handling lives in `main.rs`, and the
two are coupled by timeouts (e.g. `TIMEOUT_ALL_RETRIES` in lib.rs must stay below
`TIMEOUT_RECEIVE_CONTENT_LENGTH` in main.rs). There are TODOs in `main.rs` noting that retry-to-
another-mirror logic can't currently be triggered from the request handler because it lives in lib.rs.

### Arch-specific implementation (`src/mirror_flexo.rs`)

Concrete impls of the `lib.rs` traits for downloading Arch packages: `DownloadJob`, `DownloadProvider`
(a mirror), `DownloadOrder`, `DownloadChannel`, plus the curl `Handler` (`DownloadState`) that streams
bytes to disk and reports progress. Also: client HTTP request parsing (`Request`, `read_client_header`,
range/`resume_from` handling), latency-test scoring (`MirrorResults`, `rated_providers*`), and cache
inspection (`inspect_and_initialize_cache`).

### The growing-file mechanism

This is the core trick that lets concurrent clients share one upstream download. While a package is
being downloaded, it is written to a file that grows over time, and a sidecar **`.cfs` file** ("complete
file size") records the expected final size. Clients are served from this *growing* file:
`serve_from_growing_file` / `serve_growing_file_loop` in `main.rs` stream bytes as they arrive, blocking
when they reach the current end-of-file and resuming when more data is written, until the `.cfs`-recorded
size is reached. `GROWING_FILE_STALL_TIMEOUT` bounds how long the file may stop growing before giving up.
A second client requesting an `AlreadyInProgress` order joins by reading the same growing file rather
than opening a new upstream connection.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nroi/flexo](https://github.com/nroi/flexo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
