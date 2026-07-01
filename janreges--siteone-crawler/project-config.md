---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Working Principles

Adapted from Andrej Karpathy's four CLAUDE.md principles — read these before touching code. They target reasoning failures (wrong assumptions, over-engineering, scope creep, weak success criteria), not formatting.

1. **Think Before Coding** — *Don't assume. Don't hide confusion. Surface tradeoffs.*
   - State assumptions explicitly; if uncertain, ask rather than guess.
   - Present competing interpretations instead of silently picking one.
   - Call out inconsistencies, confusion, and tradeoffs as you find them.

2. **Simplicity First** — *Minimum code that solves the problem. Nothing speculative.*
   - No features beyond what was asked.
   - No abstractions for single-use code.
   - No "flexibility" or configurability that wasn't requested.
   - No error handling for scenarios that cannot occur.

3. **Surgical Changes** — *Touch only what you must. Clean up only your own mess.*
   - Match the surrounding style and conventions.
   - Don't refactor unbroken adjacent code that's orthogonal to the task.
   - Only remove what your change rendered obsolete.

4. **Goal-Driven Execution** — *Define success criteria. Loop until verified.*
   - Turn tasks into verifiable goals: "add validation" → "write tests, then make them pass"; "fix the bug" → "reproduce it in a test, then fix"; "refactor X" → "ensure tests pass before and after".
   - Iterate against those criteria instead of asking for constant clarification.

## Setup After Clone

```bash
git config core.hooksPath .githooks               # enable pre-commit hook (fmt + clippy + tests)
```

## Build & Test Commands

```bash
cargo fmt                                         # auto-format code (always run before build)
cargo build                                       # debug build
cargo build --release                             # release build (~11s)
cargo test                                        # unit tests + offline integration tests (~300 tests)
cargo test --test integration_crawl -- --ignored --test-threads=1  # network integration tests (crawls crawler.siteone.io)
cargo test scoring::ci_gate::tests::all_checks_pass  # run a single test by name
cargo clippy -- -D warnings                       # lint (CI enforces zero warnings)
cargo fmt -- --check                              # format check

# Browser rendering (`--browser`, chromiumoxide/CDP) is a DEFAULT feature — `cargo build`,
# `cargo test`, `cargo clippy` all include it. Build/test the lean variant (no chromiumoxide):
cargo build --release --no-default-features        # lean build, ~6 MB smaller, no browser
cargo test --no-default-features                   # tests without the browser feature
cargo clippy --no-default-features -- -D warnings  # lint the lean variant
```

## Quick Run

```bash
./target/release/siteone-crawler --url=https://example.com --single-page
./target/release/siteone-crawler --url=https://example.com --output=json --http-cache-dir=  # no cache
./target/release/siteone-crawler --html-to-markdown=page.html                               # convert local HTML to markdown (stdout)
./target/release/siteone-crawler --html-to-markdown=page.html --html-to-markdown-output=page.md  # convert to file
```

## Architecture

### Crawl Lifecycle (in order)

1. **CLI Parsing** (`Initiator` → `CoreOptions::parse_argv()`): Parses 120+ CLI options, merges config file if present, validates. Exits with code 101 on error, code 2 on `--help`/`--version`. Non-crawl utility modes (`--serve-markdown`, `--serve-offline`, `--html-to-markdown`) exit early in `main.rs` before creating the Manager.

2. **Analyzer Registration** (`Initiator::register_analyzers()`): Creates all 17 analyzer instances (Accessibility, BestPractice, BrowserConsole, Caching, ContentType, DNS, ExternalLinks, Fastest, Headers, Page404, Redirects, Security, SeoAndOpenGraph, SkippedUrls, Slowest, SourceDomains, SslTls) and registers them with `AnalysisManager`. Some analyzers receive config from CLI options (e.g. `fastest_top_limit`, `max_heading_level`); `BrowserConsole` is active only in `--browser` mode.

3. **Manager Setup** (`Manager::run()`): Creates `Status` (result storage), `Output` (text/json/multi), `HttpClient` (with optional proxy, auth, cache), `ContentProcessorManager` (HTML, CSS, JS, XML, Astro, Next.js, Svelte processors), and the `Crawler` instance. The crawl loop fetches through an `Arc<dyn Fetcher>` (`src/engine/fetcher.rs`): by default the `HttpClient`; with the `browser` feature + `--browser`, a `BrowserRenderer` that renders each HTML page in Chromium and returns the same `HttpResponse` (plus `browser_diagnostics`). Everything downstream is unchanged.

4. **Robots.txt Fetch** (`Crawler::fetch_robots_txt()`): Before crawling starts, fetches and parses `/robots.txt` from the initial domain. Respects `--ignore-robots-txt` option.

5. **Crawl Loop** (`Crawler::run()`): Breadth-first concurrent URL processing:
   - URL queue (`DashMap`) seeded with initial URL

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [janreges/siteone-crawler](https://github.com/janreges/siteone-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
