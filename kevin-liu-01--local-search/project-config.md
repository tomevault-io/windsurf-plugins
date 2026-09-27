---
trigger: always_on
description: Use this file as the repository-level operating guide. Read the nearest nested
---

# local-search Agent Guide

Use this file as the repository-level operating guide. Read the nearest nested
`SKILL.md` before editing a directory that has one. Use the root `SKILL.md` for
the complete end-user `lsearch` command surface.

## Product in one sentence

`local-search` is a small, open-source local browser API for shell-capable agents.
Through one Rust CLI, agents can search, read, extract, interact, and use sessions
in a dedicated Chrome or Chromium profile—without a hosted browser service or a
separate SDK for every site.

The Cargo package is `local-search`. The preferred executable is `lsearch`.
`local-search` and legacy `local-browser` remain compatibility binaries.

## Non-negotiable product principles

1. **The local browser is the product boundary.** Browser work happens in a
   dedicated profile on the user's machine. local-search is the bridge between
   an agent command and that browser, not a hosted service or transparent network
   tunnel. Do not replace it with a hosted browser or paid search API dependency.
2. **Agent output is compact and stable.** Preserve structured stdout, stable
   JSON field names, useful error codes, and low-noise stderr. Treat output shape
   changes as public API changes.
3. **Human and machine views are separate presentations.** Interactive terminals
   may use color, progress, summaries, and clickable links. Pipes and `--json`
   must remain deterministic and easy for agents to parse.
4. **Use the least browser authority needed.** Prefer high-level commands over
   arbitrary evaluation. Redact cookie values by default. Never leak browser
   credentials, tokens, request bodies, or signed-in page content into fixtures,
   logs, screenshots, or commits.
5. **Managed Chrome is the stable default.** Prefer `lsearch launch` with its
   dedicated persistent profile. Keep explicit CDP attachment for users who
   intentionally choose it.
6. **Automation must not steal focus.** Create background targets for searches,
   reads, and temporary requests. Do not make repeated agent searches pull the
   active window away from the user.
7. **Native and small beats another runtime.** Keep the Rust dependency graph and
   release binary lean. Avoid adding dependencies for convenience when a small,
   clear implementation is practical.
8. **Claims require reproducible evidence.** Performance, token, reliability,
   size, and provider-comparison claims must match committed benchmark output and
   documented methodology.
9. **Compatibility is intentional.** Keep the three binary entry points, legacy
   environment aliases, and documented output contracts unless a breaking change
   is explicitly approved.

## Repository map

### Rust CLI

- `src/bin/lsearch.rs` is the primary executable entry point. It parses Clap,
  dispatches commands, and emits structured errors.
- `src/bin/local-search.rs` and `src/bin/local-browser.rs` are compatibility
  shims. Keep them behaviorally identical to `lsearch`.
- `src/cli.rs` defines the public command-line schema: global flags, commands,
  arguments, enums, defaults, and help text. A change here is user-facing.
- `src/commands/mod.rs` orchestrates command behavior. Keep transport details and
  evaluated browser programs out of this layer when they belong under `browser/`.
- `src/browser/discovery.rs` resolves explicit CDP endpoints, managed profile
  markers, saved endpoints, and supported local ports.
- `src/browser/cdp.rs` is the minimal flattened-session Chrome DevTools Protocol
  client. It owns websocket request/response/event plumbing and target sessions.
- `src/browser/scripts.rs` contains deterministic JavaScript evaluated in pages
  for snapshots, search normalization, readable extraction, interactions,
  mapping, and authenticated requests.
- `src/config.rs` owns config/cache/profile/PID/endpoint paths and local state.
- `src/output.rs` owns stable success and error envelopes.
- `src/ui.rs` owns interactive human presentation: welcome text, progress,
  colors, hyperlinks, and ranked search rendering.
- `src/error.rs` defines stable error categories. Prefer adding a meaningful code
  over returning opaque prose.
- `tests/cli.rs` covers public CLI and output behavior.

### Documentation and maintenance

- `SKILL.md` is the complete agent-facing usage guide for installed `lsearch`.
- `README.md` is the human-facing project and crates.io documentation.
- `SECURITY.md` defines the trust model and disclosure guidance.
- `npm/localsearch/` is the npm distribution bridge. It installs an explicitly
  pinned crates.io release into a package-local Cargo root and exposes Node
  launcher aliases; it must not reimplement browser behavior in JavaScript.
- `scripts/` contains thin maintenance wrappers; it is not an alternate runtime.
- `.agent-docs/` configures generated repository documentation. Do not hand-edit
  text inside the auto-maintained marker blocks below.

### Benchmarks

- `benchmarks/README.md` documents methodology and how to reproduce comparisons.
- `benchmarks/hosted_search_benchmark.py` runs the hosted-provider comparison.
- `benchmarks/results/` holds committed evidence used by the site and README.
- Keep benchmark secrets in ignored `.env.bench.local`; never commit provider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kevin-Liu-01/Local-Search](https://github.com/Kevin-Liu-01/Local-Search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
