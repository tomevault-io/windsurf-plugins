---
trigger: always_on
description: Modern terminal file viewer. Syntax highlight, structured-data pretty-print, image render.
---

# peek

Modern terminal file viewer. Syntax highlight, structured-data pretty-print, image render.

**Single-file viewer.** One path (or stdin) at a time. No batch, no file list, no `cat`-style
concat — those belong to other tools.

## Build & Run

```sh
cargo build --workspace      # debug, all crates
cargo build --release        # release
cargo run -- [args]
cargo test --workspace       # ALL tests — bare `cargo test` runs only the bin's
cargo clippy --workspace     # ALL crates — bare skips member crates
```

**Always `--workspace`** for test/clippy. Without it cargo targets only root `peek` bin and
skips library-crates.

**Verify against the debug build** — `cargo run -- [args]` or `target/debug/peek`. Release builds
take minutes; never `cargo build --release` just to check a change. Reserve release for shipping.

No external runtime deps. Image render built in. PDF via Pdfium (ships beside binary, loaded
dynamically at startup). Ghostscript used if on path.

## Architecture map

Top-level only. Per-file detail (what + *why*) lives in each file's `//!` header — read it when
unsure where logic lives. No separate map doc.

Cargo workspace: 5 library crates under thin `peek` bin. **Layering Cargo-enforced**: detect +
parser layer barred from naming the bin's session layer (compose/gather/extract dispatch hubs +
event loop), so a bug parsing hostile bytes can't reach process/terminal control.

```
crates/
  peek-io/          input foundation. InputSource (File/Memory/FileRange/TempFile) + ByteSource +
                    LineSource (streaming, anchor-indexed) + ByteStream; single-stream codecs
                    (gz/bz2/xz/zst/lz4/br); stdin + /dev/tty reopen; limits (memory budget
                    classes). Depends on nothing in-tree.
  peek-detect/      file-type detection. FileType + per-type format enums + magic/extension/
                    content-sniff (detect.rs) + mime + transparent decompress-redetect. One
                    types/<type>.rs per type. Depends on peek-io only — NOT readers.
  peek-theme/       theming leaf. PeekTheme roles + paint; PeekThemeName + embedded .tmTheme
                    (themes/); StyleMode + SGR encode/tokenize + ActiveStyle; ThemeManager.
                    Depends on nothing in-tree. Named `peek_theme` everywhere.
  peek-foundation/  reader/viewer toolkit + info base. Above theme/io/detect, below peek-types;
                    barred from bin. Names lower crates directly. One `pub use peek_theme as theme`
                    only so `#[derive(InfoView)]` paths resolve. `testing` feature exposes test
                    helpers (off in release).
    viewer/         Mode trait + ModeId + RenderCtx + ExtractTarget; modes/ (shared modes: content/
                    pretty_view/gutter/hex/info/about/help/rendered_text<R>); listing/ (ListingMode over
                    ListSource — TreeListSource for TOCs, directory listing); table/ (TableMode +
                    RowsTableMode via RowSource); ui/ (Action/ScreenBuffer/Prompt/styled/status/
                    term); image_render (ImageConfig/ImageMode/zoom/scroll/ZoomPanState); paged
                    (PagedImageMode<R>); search; wrap_scroll; cell_size; highlight; logo_anim.
                    NB: compose_modes/ViewerState/event loop are NOT here — bin's session layer.
    info/           FileInfo + InfoExtras trait + Extras; render/ (trait dispatch, themed) + json
                    (--info --json) + time fmt; section (InfoNode tree [Row|Line|Block] + InfoView
                    + render_info + Value [Size/Count/Timestamp/Text/Split/…] + Role); rows (InfoRow
                    runtime model). 3 build modes: **derive** (`#[derive(Serialize,InfoView)]`, one
                    struct → both outputs), **InfoRow** (irregular row-shaped: cert, font),
                    **bespoke** (hand InfoView + Serialize). Display Block ≡ JSON sub-object.
    output/print    PrintOutput (write-once stdout for --print/pipes/--info) + logo painter.
    extract         Extracted / ExtractOptions / ExtractError + path sanitiser + key helpers.
    base64, xml     shared base64 + XML attr-unescape.
    derive/         `peek-foundation-derive` proc-macro for `#[derive(InfoView)]`. Walks view
                    struct → InfoNode tree: `#[info(label)]`→Row, `#[info(nest)]`→sub-view,
                    `#[info(skip)]`→JSON-only; title via `#[info(title|title_from)]`. Skips mirror
                    serde. Paths resolve only via `::peek_foundation`. Build-time only.
  peek-types/       per-file-type readers, one module per type (reader + info + view-mode; format
                    enum + sniff live in peek-detect, re-exported). Depends on foundation/detect/
                    io/theme — barred from bin. Foundation re-exported as crate::{base64,extract,
                    info,output,viewer,xml}. Owns parser deps (object, cafebabe, rusqlite, pdfium,
                    calamine, symphonia, ttf-parser, fontdue, mail-parser, x509-parser, …).
                    Types: binary, text, markdown, notebook, sql, sqlite, css, structured (JSON/
                    YAML/TOML/XML), csv, spreadsheet, presentation, image, html, email, ebook,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thaapasa/peek](https://github.com/thaapasa/peek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
