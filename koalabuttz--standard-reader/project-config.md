---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## Project

`standard-reader` (binary **`sr`**) is a **TUI reader for [standard.site](https://standard.site)** — long-form writing published to the AT Protocol (Leaflet, Pckt, Offprint, GreenGale, and any blog emitting `site.standard.*` records). Sign in with an atproto account, pull your subscriptions, and read — with images and real formatting, online or **offline**. (RSS support is a later goal.)

**Ethos (load-bearing, mirrors davidlewis.xyz): a small, justified dependency surface.** No build step beyond `cargo`; no runtime services. Prefer pure-Rust crates; every dependency should earn its place (the AVIF C decoder and an openssl TLS stack were both turned down on these grounds).

## Architecture — portable core, swappable frontend

A Cargo workspace. The split exists for **portability**: a PS Vita frontend is a stated future goal, so the engine must not assume a desktop.

```
crates/
  standard-core/   lib · ZERO platform deps · SYNCHRONOUS · the whole brain
    model            · RichDoc AST + Document/Publication/Subscription
    decode           · ContentDecoder trait + Registry + per-publisher decoders
    atp              · AtUri parsing + XRPC request building (over a Transport)
    store            · the Store cache trait
    search           · inverted index over textContent
  standard-tui/    bin `sr` · the desktop frontend (ratatui + reqwest + redb + OAuth)
```

**Two traits are the seam** — the only things a new platform implements:

- **`atp::Transport`** — perform an XRPC GET/POST and attach auth. Desktop impl: `reqwest`. A Vita impl: the Vita's net stack. The core *builds* every request URL and *parses* every response; it never opens a socket.
- **`store::Store`** — the offline cache (documents, read-state, blobs, sync cursors). Desktop impl: `redb`.

**Hard rule: never let `tokio`, `reqwest`, `redb`, `ratatui`, or `async` into `standard-core`.** The core is synchronous. The desktop frontend gets non-blocking fetches by running core operations on a worker thread and channeling results into the `ratatui` render loop; a Vita frontend calls core inline. Auth is also a frontend concern (the Vita would likely use an app-password instead of the loopback flow).

Pipeline: **`atp`** builds/parses XRPC → **`decode`** maps each publisher's `content` lexicon to one `RichDoc` → **`store`** caches it for offline → **`search`** indexes `textContent`.

## Content decoding (validated against real records)

`site.standard.document.content` is an **open union** — each publisher embeds its own lexicon. `textContent` is flat plaintext (the spec says it carries *no* formatting), so it is a **fallback only**. Decoders dispatch on `content.$type` and all target the one neutral `RichDoc` AST:

Shapes below were validated against **live records** (the published survey had several wrong field names). All decoders are ✅ implemented and tested against fixtures in `crates/standard-core/tests/fixtures/`:

| `content.$type`                          | shape                                            | decoder      |
| ---------------------------------------- | ------------------------------------------------ | ------------ |
| *(bare string)* / `at.markpub.markdown`  | Markdown (GreenGale body, Sequoia, markpub)      | `Markdown` (pulldown-cmark) |
| `pub.leaflet.content`                    | `pages[].blocks[].block` + byte-range facets     | `Leaflet`    |
| `blog.pckt.content`                      | `items: [blog.pckt.block.*]` (large → in a blob) | `Pckt`       |
| `app.offprint.content`                   | `items: [app.offprint.block.*]` + byte-range facets | `Offprint` |
| `org.wordpress.html`                     | `{ html }` — rendered HTML (`tl` walker)         | `Wordpress`  |
| `at.unthread.content`                    | `{ content }` — a Markdown string (Unthread)     | `Unthread` (reuses `from_markdown`) |
| `*#contentRef`                           | **reference** to another record (GreenGale)      | `content_ref` → two-phase |
| *(unknown / absent)*                     | typeset `textContent`                            | `Plaintext`  |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koalabuttz/standard-reader](https://github.com/koalabuttz/standard-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
