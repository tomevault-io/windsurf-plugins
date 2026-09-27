---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## The rule that outranks the others

**A registered domain must never be reported `AVAILABLE`.** Everything else in
`ds` is a convenience; that one answer is the product. A lookup that cannot be
answered is `UNKNOWN` with the reason attached, never a guess. Under-claiming is
the correct failure mode everywhere — `--where` names a registrar only where
that registrar publishes a price, and `eligibility.json` states who may buy a
name only where a registry page says so.

Any change to `src/whois.rs`, `src/rdap.rs`, `scripts/whois_classify.py`, or a
needle in `whois.json` is judged against that rule first. "It works for the TLD
I tried" is not enough — say what a wrong answer would look like and why the
change cannot produce one.

## Commands

```sh
cargo build                      # debug build
cargo run -- apple --tld com     # run it
cargo run -- --serve             # the HTTP API on http://127.0.0.1:8080
```

Before opening a pull request — all of these run offline:

```sh
cargo test
cargo clippy --all-targets
cargo fmt
cargo build --no-default-features          # the CLI without the HTTP API
python3 scripts/test_whois_classify.py     # the harvest script's classifier
node scripts/test-tld-facts-parse.mjs      # the IANA root database parser
man ./ds.1                                 # preview the manual page
```

A single Rust test: `cargo test <name>` (e.g. `cargo test harvest_script_classifier_is_in_sync`);
one module's tests: `cargo test whois::`. There are no integration tests — every
module carries its own `#[cfg(test)]` block, and tests must not need a network.

**CI does not run any of this.** The workflows in `.github/workflows` build the
site and cut releases, nothing else, so a red test only surfaces when somebody
runs it locally.

The site is a separate Astro project (Node 22+):

```sh
cd site && npm install && npm run dev
```

`npm run gen` regenerates the site's content without building — the quick way to
check a docs change.

## Architecture

A single Rust binary (`src/main.rs`, `mod` declarations at the top) plus an
Astro site and a set of Node/Python harvesters that generate the bundled data.

**The lookup pipeline** lives in `check()` in `src/main.rs`, and reading it is
the fastest way to understand the tool. Per domain, in order:

1. **RDAP** (`src/rdap.rs`) using the server list from the IANA bootstrap
   (`src/bootstrap.rs`, fetched from `data.iana.org` and cached for a week under
   `$XDG_CACHE_HOME/ds/rdap-dns.json`).
2. **WHOIS** (`src/whois.rs`) on port 43, only as a fallback or when the raw
   record was asked for. The bundled server from `whois.json` is tried first; if
   it is gone or says nothing useful, IANA is asked who serves that TLD today,
   and an answer from that referral wins and clears earlier notes.
3. **Registrability** (`src/private.rs`) — a brand or reserved TLD is `PRIVATE`,
   not `AVAILABLE`, because "no such domain" there is not an offer. This step
   runs last so the IANA referral path cannot clear its note.

Every path funnels into `CheckResult` in `src/model.rs`; `--json`, the plain
output in `print_result()`, and the HTTP API all serialize that one type.

**Pacing is per-host, not global** (`src/limit.rs`). Hundreds of TLDs share a
handful of servers — Identity Digital alone runs ~250 gTLDs — so a global
concurrency cap would still get a `--tld all` run 403'd. The `HostLimiter` caps
concurrency per host, widens the gap between requests when a host pushes back,
and trips a circuit breaker after 6 consecutive refusals. Do not raise
concurrency to make a run finish sooner; the same restraint applies to the
harvest scripts, which are deliberately paced and cache under
`scripts/.iana-cache/` and `scripts/.whois-cache/` (both gitignored).

**`--serve`** (`src/serve.rs`) ships in the default build behind the `serve`
Cargo feature. A `ds` server is an open proxy onto other people's registries, so
it is loopback-only by default, caps how much one request may ask for, shares a
single `HostLimiter` process-wide, rate-limits per client, and caches responses.
`--no-default-features` drops it and builds the CLI alone.

**Data files are embedded with `include_str!`** — `whois.json` (`src/tlds.rs`),
`pricing.json`, `private-tlds.json`, `eligibility.json` — so the binary is
self-contained. Each can be overridden at runtime by a file of the same name in
the working directory or `$XDG_CONFIG_HOME/ds/`; `discover_config()` in
`src/main.rs` resolves that, and a loaded override is always announced so a
stray file cannot quietly change results. Growth in these files is binary
growth: going from one registrar to three took `pricing.json` from 94 KB to
299 KB and the stripped release build from 4.18 MB to 4.38 MB.

**The Rust and Python WHOIS classifiers are kept in sync by a test.**
`scripts/refresh-whois.py` decides which harvested servers are safe to bundle by
running a Python port of `classify()`. `harvest_script_classifier_is_in_sync` in
`src/whois.rs` parses `scripts/whois_classify.py` and compares the marker
tables — edit one and you must edit the other.

## Data files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AminulBD/ds](https://github.com/AminulBD/ds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
