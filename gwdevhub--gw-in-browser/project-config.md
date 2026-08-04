---
trigger: always_on
description: Context for anyone — human or agent — working on this repo. `README.md` is the
---

# CLAUDE.md

Context for anyone — human or agent — working on this repo. `README.md` is the
user-facing guide; `docs/internals.md` is the technical write-up. This file
covers how the project is laid out, the conventions it follows, and the
constraints that are not obvious from reading the code.

## What this is

Guild Wars Reforged shipped on Android and iOS in June 2026. The mobile client
is not a native port: it is a Capacitor WebView wrapping an Emscripten build of
the game, with an Astro launcher around it. Every platform service is injected
onto a plain JavaScript `Module` object, so the same WebAssembly module runs in
an ordinary browser given a host that supplies those services.

`harness/` is that host. `gw.py` downloads the client, serves it, bridges its
network access, and opens a browser.

## Status

The client downloads, boots, and reaches ArenaNet's servers. It resolves all
twelve `File*.ArenaNetworks.com` content servers and exchanges 21 bytes out /
32 back with each, and it reaches `Auth1.ArenaNetworks.com` with an 82-out /
22-back handshake.

**Authentication is not implemented.** `login`, `secureStorage` and
`nativeAccount` are logging stubs, so nothing can present credentials. That is
the blocker on actually playing.

**Rendering is unverified.** Headless Chromium aborts at `GL ES 3.0 default
vertex shader compilation failed` (`Engine/Gr/Gles3/GlShaderCache.cpp:959`)
under SwiftShader, immediately after logging "first frame presented" — so the
presentation path is wired, but what a real GPU does is untested. Reports from
real hardware are the single most useful contribution right now.

## Layout

| | |
|---|---|
| `gw.py` | everything at runtime: update, serve, relay, prefetch, browser |
| `harness/` | `index.html` + `harness.{css,js}` + `loading.{css,js}` |
| `images/`, `fonts/` | loading-screen art and typeface |
| `gwkey.py` | extracts the client access key from an APK |
| `gwpatch.py` | the patch protocol, standalone |
| `getsnapshot.py`, `tools/` | analysis: wasm patching, scanning, symbol recovery |
| `tests/` | four suites, all offline against fixtures |

## Style

Applies to `gw.py` and the JS in `harness/`.

- **Single-line comments.** No block comments. A point that needs a paragraph
  belongs in `docs/internals.md`, not above the code.
- **Comment the surprise, not the mechanism.** Why something is done the odd
  way, not what the next line does. Most lines need no comment.
- **Inline one- and two-line functions.** A name is earned by being called
  from more than one place, or by holding real logic.
- **Fewer lines, same clarity** — right up to the point where a reader has to
  reconstruct intent.

## Things that will bite you

These are non-obvious properties of the client or the environment. Each one is
load-bearing; changing the code around them without knowing they exist tends to
produce a failure that looks like something else.

### Constants use non-canonical, zero-padded LEB128

`i32.const 0x102820` encodes as `41 a0 d0 c0 80 00` — five bytes, not the
canonical four — because LLVM emits fixed-width relocatable encodings. Anything
searching for constants must **decode**, not byte-match an encoded needle:
searching for the canonical form finds nothing, silently, and looks exactly
like "this value is never referenced".

### Nine host functions are awaited

`image.cacheAsync`, `dns.resolve`, all three of
`secureStorage.{get,store,clear}Credentials`, `adProvider.showInterstitial`,
`ageSignals.check`, `shop.initialize`, `shop.inAppPurchase`. Returning
`undefined` from one throws `Cannot read properties of undefined (reading
'then')` and kills the frame mid-connect, so a stub must be a promise, not
merely callable. Re-derive the list if the client updates: find every
`Module.x.y(...)` that is `await`ed or `.then()`d.

### `Module` must be `var`

The glue opens with `var Module = typeof Module != 'undefined' ? Module : {}`.
A `const`/`let` in the harness collides at parse time, or is invisible and the
glue silently builds its own empty object. `tests/test_harness.js` covers both
failure modes.

### `image.fileSize` is synchronous

So the snapshot size must be known before the glue loads, which is why the
harness reads `snapshot-chunks.json` first.

### Both builds share an output basename

`Gw.jspi.js` asks for `Gw.wasm` exactly as `Gw.js` does. `locateFile` has to
redirect, or the JSPI glue silently pairs with the Asyncify binary whenever
both files are present.

### Concurrent chunk reads must be deduplicated

The module drives `cacheAsync` over overlapping regions with ~160 requests in
flight. Without one shared promise per chunk it fetches the same chunk several
times over — measured at 4x amplification, enough that a boot never finished.
See `chunkBytes()` in `harness/harness.js`.

### Snapshot reads are latency-bound, not bandwidth-bound

Each 256 KB chunk costs ~650 ms from the CDN, so throughput scales almost
linearly with concurrency. `PREFETCH_JOBS` is capped at 8 on purpose; see
**Conduct** below before raising it.

### Browsers throttle hidden tabs

Chrome drops background tabs to roughly one timer tick a minute. Anything that
treats a missing heartbeat as "gone" needs a tolerance well above that, or it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gwdevhub/gw_in_browser](https://github.com/gwdevhub/gw_in_browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
