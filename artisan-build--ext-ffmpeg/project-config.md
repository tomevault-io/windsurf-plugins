---
trigger: always_on
description: generates the 475 `FFmpeg\Filter\*` classes + 395 enums into `packages/ext-ffmpeg-filters/`
---

# ext-ffmpeg — working notes for Claude

Native PHP extension binding the FFmpeg libraries. **Pure C against the Zend API** —
no PHP-CPP, no FFI, no Rust, no `extern "C"` (both languages are C). Built on FFmpeg;
credit it, don't imply affiliation.

## Read first
- `docs/decisions/0002-foundation-naming-scope.md` — why C, naming, scope, workspace.
- `docs/decisions/0003-v1-api.md` — the v1 API (Media / streams / MediaEncoder).
- `docs/decisions/0004-project-structure-and-dx.md` — concern dirs, one stub per class,
  stub-first codegen. `docs/contributing/` is the how-to (build, structure, add-a-class).
- `docs/decisions/0001-no-private-struct-access.md` — **never** reach into FFmpeg's
  private structs (the PoC did; it was unsafe). Custom AVFilter on the fork instead.
- `docs/decisions/0005-licensing-and-trademark.md` — **OPEN/unresolved**: no LICENSE chosen
  yet; static-bundled FFmpeg ⇒ the released binary is **GPL** — must be settled (+ trademark)
  before any public release.
- `docs/decisions/0006-linking-and-distribution.md` — released builds **statically bundle
  FFmpeg** into `ffmpeg.so` (self-contained; required for hosts like Laravel Cloud). Dev
  builds stay shared. The bundled FFmpeg version is the feature ceiling.
- `docs/PRD.md` — the long-range vision. `docs/ffmpeg-notes.md` — FFmpeg gotchas.
- `site/` — the docs website (Astro + Starlight; an Artisan Build project). User-facing
  reference lives at `site/src/content/docs/` (mirrors the namespace); filter pages use
  `site/src/content/docs/filters/_TEMPLATE.md` — the crown-jewel doc artifact. See
  `docs/contributing/website.md`. **How to write a filter page's prose** (voice, section spec,
  verify-every-example rule, assets — built to drive an eventual per-filter authoring loop) lives in
  `docs/contributing/filter-page-authoring.md`; `scale` is the reference exemplar.

## Workspace
This repo is also the dev workspace. `php-src/` and `ffmpeg/` are gitignored sibling
**forks** (origin = ProjektGopher, upstream = the real project), cloned via
`make bootstrap` at the refs in `manifest.toml` (php PHP-8.5, ffmpeg n7.1.1).

Build/dev against our own from-source FFmpeg (Homebrew's 8.0 dropped freetype/drawtext)
and our own **debug + ASAN/UBSAN** php-src (turns segfaults into stack traces).

## Commands
```
make bootstrap   # clone the forks (once)
make ffmpeg      # build FFmpeg -> ffmpeg/_install (freetype/harfbuzz/etc enabled)
make php         # build php-src debug+ASAN -> php-src/sapi/cli/php
make stubs       # regenerate src/**/*_arginfo.h from the .stub.php files (commit them)
make ext         # phpize + build ffmpeg.so against the dev php + dev ffmpeg
make test        # run tests/*.phpt against the dev php
make clangd      # unified compile_commands.json (needs `bear`) for cross-repo go-to-def
```
`make stubs` runs `php-src/build/gen_stub.php` with the **dev php** — we build `tokenizer`
+ `ctype` into it (gen_stub/PHP-Parser need them; `json` is always in), and gen_stub vendors
PHP-Parser under `php-src/build/` on first run. So codegen needs **no Homebrew/Herd php**.
Generated `*_arginfo.h` are committed, so `make ext` never needs the codegen toolchain. Only
run `make stubs` after editing a `.stub.php`. (tokenizer/PHP-Parser are also groundwork for
the planned PHP-in-filter-expression feature.)
Run the ASAN php with `USE_ZEND_ALLOC=0 ASAN_OPTIONS=detect_leaks=0` (LSan is
unavailable on macOS/arm64, so address+UB only — no leak detection).

Smoke test: `make smoke` (probes the fixture; also exercises a thrown exception).

Layout: the extension is self-contained in `ext/` — its own phpize build dir. `ffmpeg.c`
is thin (module entry + MINIT dispatch); each class lives in its own concern under
`ext/src/` (`exception/ media/ stream/ codec/ encoder/`, later `filter/`). **One
`.stub.php` per class** drives gen_stub; see ADR 0004 + `docs/contributing/structure.md`.
The orchestration `Makefile` stays at the workspace root so `phpize --clean` can't delete
it. `php-src/` and `ffmpeg/` are gitignored forks.

`phpize --clean` (run by `make ext` on every build) wipes **`tests/*.php`** — it treats
them as generated `.phpt` scratch. So the hand-written manual smoke script lives at
`ext/probe.php`, NOT `ext/tests/`. Keep `tests/` to `.phpt` + `fixtures/` only.

## Object model (the load-bearing pattern)
- `Media` owns the input `AVFormatContext`; freed in `free_obj` (the ONE native teardown
  that must be right). Probe is eager (`open_input` + `find_stream_info`).
- `VideoStream`/`AudioStream` are non-owning views; each `GC_ADDREF`s its parent `Media`
  and `OBJ_RELEASE`s it in `free_obj`. They borrow `AVStream`, own nothing.
- `MediaEncoder` is output/mux: composed from streams via typed `addVideo`/`addAudio`,
  `save($path)` is stateless (build→run→teardown output ctx in the call). Holds zvals to
  streams (which pin their Media), no long-lived native memory. Re-encode path is
  decode→(sws / swr+AVAudioFifo)→encode; `Copy` is packet-level stream-copy/remux.

## Current state
Implemented: `Media::open`, `->duration/->format/->bitrate`, `->streams()`,
`->videoStream()/->audioStream()`, the stream view classes, the `FFmpeg\Exception\*`
hierarchy, and **`MediaEncoder`** (`addVideo`/`addAudio`/`save`) with the string-backed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artisan-build/ext-ffmpeg](https://github.com/artisan-build/ext-ffmpeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
