---
trigger: always_on
description: Guidance for working in this repository. This is the authoritative rule source —
---

# CLAUDE.md

Guidance for working in this repository. This is the authoritative rule source —
when README, code comments, or memory disagree with this file, this file wins, and
the disagreement should be fixed.

typort is a **universal Typst → Word (`.docx`) converter**. Any valid `.typ`
should convert to an editable `.docx`. For how it works, read
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) first.

## Philosophy (the rules that decide design)

1. **Universal, not genre-specific.** typort converts *any* `.typ`, in *any*
   language. Never hardcode assumptions about a document's genre or language —
   no "this is a Chinese social-science paper" logic, no matching natural-language
   keywords (`参考文献`, `表`, `图`, `References`, `Abstract`, …) to drive behavior.
   When you need to identify a construct, use the **semantic Typst element**
   (`BibliographyElem`, `FigureElem`, `FootnoteElem`, caption supplement metadata)
   or a declared source value (`#set text(lang: ...)`), not rendered text.
   `convert/bibliography.rs` is the model to copy.

2. **Detect, don't assume.** Styling (fonts, sizes, colors, spacing, alignment,
   margins) is read from the actual rendering or the source AST — never
   hardcoded for a document we happened to test with. Ask "does this work for
   *any* Typst document?", not "does this work for my fixture?".

3. **Semantic-first, geometry-as-fallback.** Prefer HTML semantics + the
   introspector (which preserve "this is a heading / footnote / equation"). Fall
   back to Paged geometry only for things HTML cannot express. Geometry inference
   is the lossy path we exist to avoid — keep it contained.

4. **Before declaring something impossible, check all three sources.** If Typst
   renders it correctly, the information exists *somewhere* — a "can't" almost
   always means "not in the representation I happened to look at." Elements consumed
   during compilation are queryable in neither the HtmlDocument nor the
   PagedDocument (introspector hits = 0), yet are still present in the **source AST**:
   `#colbreak()`, `smallcaps`, `#set text(lang:)`, and `datetime.today()`'s inputs
   are all recovered from source, not from the compiled output. Probe (compile +
   query/parse) before concluding, and look for a precedent — smallcaps' source-AST
   recovery was the template for colbreak. Say "I haven't found a way" rather than
   "there is no way" until you have proof there is no principled rule.

## Architecture in one paragraph

typort compiles the same source to **`HtmlDocument`** (semantics + document order
+ introspector) *and* **`PagedDocument`** (fonts, geometry, images, layout-only
content), and additionally **re-parses the source AST** for authoritative `set`
rules. HTML is the skeleton; Paged paints and patches it; the AST overrides both
when the author declared a value. This is **three** sources, not two — don't let
anyone "simplify" it back to a dual-compilation description. Full detail in
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## Rust conventions

- **Edition 2024**, `max_width = 100` (`rustfmt.toml`). Run `cargo fmt` — CI fails
  on unformatted code.
- **`#![warn(clippy::pedantic)]`** per crate; **CI runs clippy with `-D warnings`**
  (`--all-targets`). Zero warnings is a hard gate, not an aspiration.
- **`#[allow(...)]` policy:** prefer the narrowest scope (item-level over
  file/crate-level) and pair each with a one-line justification. The legitimate
  bulk today is `cast_possible_truncation` / `cast_sign_loss` in layout math
  (intentional `f64 → twips/half-point`). Treat `too_many_lines` and
  `too_many_arguments` allows as **refactor signals**, not solutions (see below).
- **`unwrap()` / `expect()` / `panic!`:** this tool ingests arbitrary user input.
  Don't `unwrap` on parsed or `Option` data derived from the document.
  `unwrap()` on infallible in-memory writes (e.g. `quick-xml` to a `Vec<u8>`) is
  tolerated but prefer a documented helper. No `todo!`/`unimplemented!` in `src`.
- **Argument threading:** shared walk state is bundled in context structs
  (`WalkCtx<'a>` for the HTML walk, `InlineFmt` for inline formatting flags,
  `TableWidthCtx` for table sizing) rather than positional parameters. The
  former `clippy.toml` `too-many-arguments-threshold` override was removed once
  that debt was repaid — new shared state goes into one of these structs, not a
  new positional parameter.
- **File size:** don't grow files reflexively. The HTML walk is split by
  responsibility under `convert/` (`block`, `inline_walk`, `headings`, `tables`,
  `lists`, `source`, `smallcaps`, `postprocess`, and `dom`); paged-style
  extraction is split under `convert/page/` (`units`, `style`, `source_ast`,
  `hanging_indent`, `reachable`, `sections`, `margin`, `run_style`, and
  `language`); recovery is split under `convert/recovery/` (`lines`,
  `deduplication`, `insertion`, `horizontal_rules`, and `table_rules`), with
  shared walk/recovery text normalisation in `convert/text_norm.rs`; and the
  OOXML writer is split by emitted part under `typort-ooxml/src/writer/`. Each
  `mod.rs` is a re-exporting facade. Put a new converter, paged-style
  responsibility, or writer part in its matching module rather than growing an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dianzuan/typort](https://github.com/dianzuan/typort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
