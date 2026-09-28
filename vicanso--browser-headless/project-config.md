---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A headless-Chrome HTTP service: POST a URL, get back a structured snapshot of
the rendered page (HTML/text/markdown, performance timings, every network
resource, JS exceptions, security audit, Core Web Vitals, optional
screenshot/PDF/HAR/DOM-snapshot). Built on **chromiumoxide** (CDP client) +
**axum** (HTTP), Rust 2024 edition.

## Commands

```bash
cargo build                 # debug build
cargo build --release       # production build
cargo run                   # run the server (Makefile: `make dev`) — needs Chrome, see below
cargo test                  # all tests (12 pure unit tests in browser.rs; no Chrome needed)
cargo test <name_substr>    # a single test by name
cargo fmt --all             # format (Makefile: `make fmt`)
cargo clippy --all-targets --all-features -- -D warnings   # the lint gate CI enforces
```

**CI (`.github/workflows/ci.yml`) gates every push/PR on three things:**
`cargo fmt --all -- --check`, `cargo clippy --all-targets --all-features -- -D warnings`,
`cargo test`. Run all three before pushing — `make lint` only runs bare
`cargo clippy` (no `-D warnings`), which is weaker than the gate.

**Running the service needs a Chrome/Chromium binary.** chromiumoxide reads
`$CHROME` first, else scans PATH. On macOS dev:
`CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" cargo run`.
The binary listens on `0.0.0.0:3000`.

The unit tests are pure functions (CSP/resource-summary/cookie parsing in
`browser.rs`) and do **not** launch Chrome — so `cargo test` passes without a
browser, but they cover almost nothing in `http.rs`/`capture.rs`/`pool.rs`. For
those layers, verify with a live smoke test: start the server and `curl` the
endpoints (`/readyz`, `/summary?url=...`, `POST /summary/batch`, `/metrics`).

## Architecture

Module layering is the thing to internalize — it was deliberately split so the
capture engine is reusable outside HTTP:

- **`main.rs`** — entrypoint only: builds the tokio runtime, then dispatches on
  `BROWSER_HEADLESS_MODE` — `serve` (default) runs the HTTP API, `worker` runs
  the Redis queue consumer (plus a health-only `/healthz`+`/readyz`+`/metrics`
  listener on `BROWSER_HEADLESS_HEALTH_PORT`, default 3000), `all` runs both in
  one process (worker as a background task) sharing one `CaptureCtx` / pool,
  `mcp` serves MCP over stdio (logs rerouted to stderr — stdout is the
  protocol channel). A `healthcheck` argv subcommand
  (`browser-headless healthcheck`) does an internal `GET /healthz` and exits
  0/1 — it's the container HEALTHCHECK, so the image needs no curl/wget.
- **`config.rs`** — env-backed knobs, each cached once (`default_timeout_ms`,
  `deadline_buffer_ms`, `max_batch_urls`, `checkout_wait_ms`, `health_port`).
- **`capture.rs`** — the **HTTP-agnostic capture core**. `CaptureCtx { pool,
  allow_private_ips }` is the only context it needs; `capture_one(&CaptureCtx,
  SummaryQuery) -> Captured` is the unit of work; `run_batch` fans out under
  the pool. Also owns `SummaryQuery` (the params DTO), the initial-URL SSRF /
  scheme pre-check (delegated to `ssrf`, fast-reject before pool checkout), and
  per-capture metrics. `capture_one` bounds the pool-slot wait by
  `checkout_wait_ms()` (admission control — 503 on saturation rather than an
  unbounded queue). **Depends only on `browser` + `pool` + `config` — never
  axum.** This is the reuse boundary: both `http.rs` and `worker.rs` call
  `capture::capture_one` without depending on each other.
- **`http.rs`** — the axum layer: `router()`, all handlers (`/summary`
  GET+POST, `/summary/batch`, `/healthz`, `/readyz`, `/metrics`), API-key auth,
  request-shape logging, and Prometheus recorder install. `AppState` embeds a
  `CaptureCtx`; `HealthState` (a `FromRef` sub-state, no API key) backs the
  probe/metrics routes, and `health_router()` exposes just those three for
  worker mode.
- **`mcp.rs`** — the MCP (Model Context Protocol) front-end via the official
  `rmcp` SDK: four agent-oriented tools (`fetch_page` with max_chars + meta
  header / `page_signals` compact JSON / `screenshot` / `page_audit`)
  that all funnel into `capture::capture_one`, so SSRF, admission control,
  clamps, and metrics apply unchanged. Two transports off one `McpServer`:
  Streamable HTTP mounted at `/mcp` by `http::router` (behind the same
  X-Api-Key check, via router-side middleware — the MCP service is a raw tower
  service and can't run handler auth), and stdio for `BROWSER_HEADLESS_MODE=mcp`.
  Depends on `capture` + `browser` + `rate_limit` — never axum. Heavy payloads
  (`resources[]`, HAR, PDF, DOM snapshots) are deliberately NOT exposed as
  tools; `page_audit` blanks `stat.data` before `to_markdown` so the report
  isn't drowned by page HTML. Disable the HTTP mount with
  `BROWSER_HEADLESS_DISABLE_MCP`.
- **`pool.rs`** — `BrowserPool`: a fixed pool of N chromium instances.
  `checkout()` routes each request to the least-loaded active instance and is
  the concurrency gate (per-instance semaphore of `pages_per_instance`; total
  concurrency = `pool_size × pages_per_instance`). Each instance has its own

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vicanso/browser-headless](https://github.com/vicanso/browser-headless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
