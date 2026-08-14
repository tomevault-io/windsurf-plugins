---
trigger: always_on
description: Operating manual for Claude Code in this repo. Strict architecture,
---

# CLAUDE.md — White Proxy

Operating manual for Claude Code in this repo. Strict architecture,
loose implementation. Read once per session, then ship.

White proxy is a **pure-Go**, cross-platform (Linux, macOS, Windows)
proxy + scanner + DPI-desync suite for circumventing TLS-based
censorship. The previous Python+Go hybrid has been retired — there is
no `cores/`, `utils/`, `api/`, or `main.py` anymore. If your training
data remembers them, ignore it. The current source of truth is the
tree under `cmd/`, `internal/`, and `pkg/`.

## Project shape (orient quickly)

- `cmd/whiteproxy/` — the **single bundled binary**. Dispatches to one
  of the inner subcommands based on `os.Args[1]`. With no args, opens
  the TUI (auto-spawning the daemon if no socket answers).
  Subcommands:
    - `daemon`         — long-lived daemon (RPC over AF_UNIX)
    - `tui`            — interactive bubbletea TUI (default)
    - `ctl`            — kubectl-style client (status, down, config,
                        events, route, scan, sub, relay subcommands)
    - `scan`           — proxy/SOCKS5/HTTP discovery scanner
    - `whitescan`      — T14 white-IP TLS-admit scanner
    - `snicheck`       — SNI accessibility probe (CLI)
    - `snicheck-tui`   — SNI probe (TUI front-end)
    - `routeverify`    — route-verify daemon (long-running, stdin/stdout)
    - `relay`          — SNI-desync TCP relay (Linux only)
- `cmd/<other>/` — historical entry points still wired through the
  dispatcher in `cmd/whiteproxy/`. `whiteproxyd`, `whitep`, `relay`,
  `scan`, `whitescan`, `sni_check`, `sni_check_app`, `routeverify`
  are all packages exposing a `Run()` rather than `main()` — the
  bundled binary calls them by name. Don't add new `package main`
  entry points; extend the dispatcher table instead.
- `internal/daemon/` — daemon glue: RPC server, job orchestration,
  proxy/relay/scan/sub method handlers. Compiler-enforced no external
  import (Go enforces `internal/` visibility).
- `internal/tui/` — bubbletea TUI. Tabs are Scan / Monitor / Routing
  / Sub / Inspect / Settings (white mode) plus Control / Test / Relay
  (desync mode, Linux only).
- `pkg/` — importable libraries. The package boundary IS the API.
    - `paths`         — XDG-style per-user dirs (Config / Cache /
                        Data); `paths.Ensure()` mkdirs them.
    - `config`        — JSON config snapshot + reload.
    - `ipc`           — JSON-RPC 2.0 over AF_UNIX (Linux/macOS/
                        Windows10+); streaming responses.
    - `eventbus`      — typed, in-process pub/sub used inside the
                        daemon and to fan out to RPC subscribers.
    - `jobs`          — job lifecycle (state, events, cancel).
    - `pause`         — cooperative pause/resume `Gate` with
                        Begin/End/Inflight counters.
    - `asn`           — embedded Iran + Global ASN databases
                        (`assets/filtered_ipv4.csv` and
                        `assets/ipv4.csv.xz`). Sorted-prefix index
                        for O(log N + small) lookups; warmable.
    - `classify`      — verdict classifier (HasBrowserBlock,
                        HardReject patterns, etc.).
    - `route_engine`  — routing + scan pool + bans + cache.
    - `proxy`         — HTTP CONNECT + SOCKS5 listener; routes via
                        route_engine; MMDF MITM domain-fronting.
    - `mmdf`          — MITM domain-fronting (CA install +
                        per-profile config).
    - `desync`        — TLS-desync (Linux only via `//go:build
                        linux`). macOS/Windows builds compile it out.
    - `sub`           — sub-server: in-process HTTP UI (port 7081)
                        + autoload of latest scan results.
    - `scan/<X>`      — scanner pipelines, all in-process library
                        calls (NOT subprocess + JSONL anymore):
        - `common`    — shared probe/Config types.
        - `targets`   — target expansion (CIDR, file, paste, ASN
                        scopes, streaming).
        - `cache`     — alive/dead IP caches (byte-compatible
                        with the legacy on-disk JSON shape).
        - `white`     — T14 white-IP TLS-admit scanner.
        - `proxy`     — HTTP/SOCKS5 proxy discovery scanner.
        - `sni`       — SNI accessibility pipeline (resolvers,
                        sinkholes, profiles, fingerprints).
        - `route`     — route-verify probes.
        - `speed`     — concurrency / wave profiles.

- `data/`, `cyclic_archives/` — runtime state (created lazily). The
  daemon defaults to `paths.DataDir()` (e.g.
  `~/.local/share/whiteproxy`) when no `--proxy-scan-dir` /
  `--proxy-log-dir` is passed, so the binary is truly standalone.
- `research/` — Obsidian-style experiment vault. **Index-first**
  navigation: `research/CHECKLIST.md` is the Map-of-Content; every
  experiment row has a `[[wiki-link]]` + one-sentence conclusion.
  See §5 below.

---

## 1. The Sandbox (Hard Rules)

These are non-negotiable. Breaking one is a regression even if tests
pass.

1. **Daemon owns long-lived state.** The proxy listener, route
   engine, MMDF CA, MMDF registry, scan job manager, relay process,
   sub server, and config snapshot all live in the daemon. The TUI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sahroush/WhiteProxy](https://github.com/sahroush/WhiteProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
