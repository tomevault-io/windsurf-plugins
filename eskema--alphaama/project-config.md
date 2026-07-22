---
trigger: always_on
description: A minimalistic Nostr client in vanilla JS. Built for the author first, no compromises for user acquisition. Exposes the protocol rather than abstracting it.
---

# Alphaama (A<3)

A minimalistic Nostr client in vanilla JS. Built for the author first, no compromises for user acquisition. Exposes the protocol rather than abstracting it.

- **Stable**: alphaama.com
- **Dev/WIP**: wip.alphaama.com (tracks repo, may have breaking changes)

## Philosophy

- Vanilla JS by design: no build step for the app itself. Learning the language deeply matters more than convenience.
- Dependencies are vendored in `dep/` as pre-built bundles. Only things that don't make sense to reimplement (crypto, protocol tooling, streaming, ratchet, OPFS store).
- Slow ES module migration: most code still hangs off the global `aa.*` namespace. `aa/regex.js`, `o/o.js`, `b/b.js`, and `av/av.js` are the only true ES modules so far.

## Conventions

- **snake_case** everywhere
- Opening brace on its own line (Allman style):
  ```js
  function foo()
  {
    // ...
  }
  ```
- Short module/directory names mirror Nostr single-letter tags (e, p, q, r, m, d, b, w) — less typing, shorter URLs
- Funny/weird names are intentional. Examples: `caralho` (home button — Portuguese for "go fuck yourself"), `tits` (title-and-state setter in `aa/view.js`), `yolo` (sign+mine+broadcast button in drafts), `dickbutt` (placeholder state text)
- Service worker file is `cash.js` at the repo root
- CLI prefix is `.` (was `.aa` historically — see commit d4d1fc6)

## Architecture

Everything hangs off a single global `aa` object. Each module extends it. Inter-module communication increasingly goes through a small **event bus** (`aa.bus`) and a **readiness gate** system (`aa.mod.ready` / `aa.mod.fire_ready`) instead of direct calls — both are framework-level facilities introduced to break circular deps and replace timing-based delays.

### Modules

| Dir | Namespace | Loaded as | What | Key files |
|-----|-----------|-----------|------|-----------|
| `aa/` | `aa.*` | scripts | Core framework: init, element registry (Map), utilities, event bus, readiness, log, view, mod loader | aa.js, bus.js, mk.js, mod.js, view.js, clk.js, log.js, parse.js, regex.js (ESM), wakelock.js, fx/ (array, color, dom, nostr, string, tags, time, util, validate) |
| `cli/` | `aa.cli` | mod | CLI input (`.` prefix, customizable), command parsing, history, autocomplete, `&&` chaining, `\|` piping | cli.js |
| `o/` | `aa.o` | mod (esm) | Options w/ Proxy-backed session overrides over localStorage (`o ss`), cross-tab sync via storage events | o.js |
| `e/` | `aa.e` | mod | Events: creation, signing, rendering, threading, orphan handling, mute system, draft editor, analytics | e.js, kinds.js, printer.js, render.js, parse.js, mk.js, miss.js, miner.js, db.js, fx.js, clk.js, anal.js, views.js |
| `p/` | `aa.p` | mod | Profiles: metadata, follows/followers, Web-of-Trust scoring, NIP-05, relay hints | p.js, mk.js, clk.js, view.js |
| `q/` | `aa.q` | mod | Queries: filter templates, variable expansion, subscriptions, outbox, NIP-A7 spells, URL-driven views | q.js, spells.js, views.js |
| `r/` | `aa.r` | mod | Relays: WebSocket mgmt via worker, scoring, batching, outbox model, NIP-42 auth, progressive backoff | r.js, manager.js (worker), msg.js, fx.js, mk.js |
| `u/` | `aa.u` | mod | User: auth, **signer abstraction** (NIP-07 + NIP-46 unified), bootstrap, decrypt cache, variable system | u.js |
| `m/` | `aa.m` | mod | NIP-17 gift-wrap DMs (rumor/seal/wrap, kind 1059) | m.js, mk.js, clk.js, view.js |
| `d/` | `aa.d` | mod | Double Ratchet DMs (Signal-style, vendored from mmalmi's nostr-double-ratchet) | d.js, mk.js, view.js |
| `b/` | `b` (esm default) | mod (esm) | Blossom: kind 24242 auth, kind 10063 server lists, BUD-01..06 | b.js, mk.js, kinds.js |
| `w/` | `aa.w` | mod | walLNut: zaps (NIP-57), NWC (NIP-47), Cashu (NIP-60/61) with AES-GCM proof vault. **Experimental** | w.js, mk.js, clk.js, fx.js, kinds.js |
| `h/` | `aa.h` | mod (esm) | Help — renders all module READMEs in a unified view; nav auto-built from modules with a prefetched `mod.readme` | h.js |
| `db/` | `aa.db` | scripts | DB orchestration: persistent IDB worker, request_id tracking, service-worker registration | db.js, idb.js, sdb.js |
| `av/` | — | dynamic ESM | Audio/video player; loaded inline by `aa.load`, exposes only `aa.mk.av` and `aa.fx.generate_waveform` | av.js |

Mod load order in `aa/aa.js`: `cli, o, p, e, m, r, q, u, b, w, d, h`. Order matters — later modules can depend on earlier ones being loaded. `o`, `b`, and `h` are loaded as ESM via `import()`; the rest are script tags.

### Module file convention

Modules typically have some subset of:
- `X.js` — core logic + `load()` init
- `mk.js` — DOM element creators (factories)
- `clk.js` — click handlers
- `fx.js` — utility functions
- `kinds.js` — Nostr event kind-specific handlers
- `view.js` / `views.js` — URL hash view handlers
- `README.adoc` — module docs (each module has one)

Vary in practice: `e/` is the biggest with 14 files; `b/` and `d/` are leaner.

### Supporting code

| Dir/File | What |
|----------|------|
| `scripts/make.js` | Element factory `make(tag, options)` |
| `scripts/sift.js` | Filtering, sorting, pagination engine. Supports keyword filters (`kind:1 by:alice text`) — see `sift.matchers` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eskema/alphaama](https://github.com/eskema/alphaama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
