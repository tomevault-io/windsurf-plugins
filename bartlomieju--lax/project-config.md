---
trigger: always_on
description: This file is for AI agents and new contributors. It captures the architecture,
---

# Working in this repo (agent guide)

This file is for AI agents and new contributors. It captures the architecture,
conventions, and workflows that aren't obvious from the code alone. Read it
before making changes.

## What lax is

A family of **lax** formatters: they never reinterpret your code, they only
adjust whitespace. The input is tokenized losslessly, structure is recognized
only where it is unambiguous, and the printer emits the same tokens with
normalized spacing. Anything unknown, dialect-specific, broken, or ambiguous
**passes through verbatim** and stays stable across passes.

This is the entire design philosophy and it is load-bearing. When in doubt:

- **Never invent or drop a token.** A semicolon the author omitted is not
  added; a token the author wrote is never removed. (Whitespace is the only
  thing that moves. One documented exception: a declaration the formatter can
  *fully see* may get a terminating separator where the grammar guarantees one,
  but an opaque/placeholder region never does.)
- **Never reinterpret.** No requoting, no case changes, no reordering, no
  rewriting values. If a region can't be understood, it is emitted as-is.
- **Degrade gracefully.** Unparseable input is not an error; it is passed
  through. Formatting must never fail on valid-but-weird input.

Two invariants follow and are machine-checked (see Testing): formatting is
**idempotent** (format twice = format once) and **content-preserving**
(nothing but whitespace changes).

## Repo layout

A Cargo workspace (edition 2024). Members:

| Crate                      | Formats                                  | Replaces      |
| -------------------------- | ---------------------------------------- | ------------- |
| `crates/lax-core`          | shared printing machinery (no formatter) | —             |
| `crates/lax-css`           | CSS, SCSS, Less                          | malva         |
| `crates/lax-sql`           | SQL (dialect agnostic)                   | sqlformat-rs  |
| `crates/lax-markup`        | HTML, XML, SVG, Vue, Svelte, Astro, ...  | markup_fmt    |

`benchmarks/` is a **workspace-excluded** crate (see `exclude` in the root
`Cargo.toml`) so its comparison dependencies (malva, sqlformat) never touch the
main build or CI. Run it explicitly with `--manifest-path benchmarks/Cargo.toml`.

These crates are consumed by `deno fmt` (the `deno` repo depends on the
published versions). See "Integration with deno" below.

## Architecture of a formatter crate

Every formatter crate (`lax-css`, `lax-sql`, `lax-markup`) has the same shape:

```
src/
  lib.rs                  # public exports (format_text, configuration)
  format_text.rs          # entry point: tokenize -> parse -> generate
  configuration/mod.rs    # Configuration struct + resolve_config()
  generation/
    tokenizer.rs          # lossless tokenizer; keeps raw &str slices
    parser.rs             # generic structure recognizer (statements/nodes)
    printer.rs            # emits PrintItems via lax-core helpers
    (keywords.rs)         # lax-sql only
  wasm_plugin.rs          # dprint Wasm plugin (gated behind `wasm` feature)
```

The pipeline in `format_text.rs::format_text_inner`:

1. **tokenize** the source into tokens that hold raw `&str` slices. The
   tokenizer is *lossless* — every byte is accounted for, comments and
   interpolations (`#{...}`, `${...}`, `@{...}`) are kept as opaque tokens.
2. **parse** generically: scan for unambiguous structure (e.g. CSS `{`/`;`/`}`,
   markup tags) and classify into statements/nodes. Unrecognized spans become
   "raw"/"verbatim" and are emitted untouched.
3. **generate** a dprint-core `PrintItems` stream, then run it through
   `dprint_core::formatting::format` with the resolved width/indent/newline
   options. `format_text` returns `Ok(None)` when the output equals the input.

### lax-core (the shared engine)

`lax-core` has no formatter of its own; it provides the printing primitives the
crates share. Key exports (`crates/lax-core/src/lib.rs`):

- `FlowPrinter` + `FlowClass` (`flow.rs`) — the heart of "lax" output. The
  printer feeds tokens to a `FlowPrinter` tagged with a `FlowClass`
  (`Whitespace { newlines }`, `Open`, `Close`, `Comma`, `CommaBreak`,
  `LineComment`, `Other`). It **preserves author newlines**, turns author
  spaces into soft wrap points (wrap at width), and indents nested
  paren/bracket groups by depth. It never introduces a break where the author
  had no whitespace.
- `push_text`, `push_text_line`, `push_comment` (`text.rs`) — emit verbatim
  text / comments into the `PrintItems` stream (comment interiors realign
  relative to their new column).
- `contains_directive`, `has_ignore_file_comment`, `HeaderToken` (`text.rs`) —
  support for `deno-fmt-ignore` / `deno-fmt-ignore-file` style directives.

If you change `FlowClass` or `FlowPrinter`, you are changing all three
formatters at once; bump `lax-core` and the crates that depend on it.

### Configuration

Each crate's `configuration/mod.rs` defines a `Configuration` struct (serde,
`camelCase`) and `resolve_config(ConfigKeyMap, &GlobalConfiguration)`.

- `Configuration` does **not** derive `Default`. Build one with
  `resolve_config(Default::default(), &GlobalConfiguration::default())` (this is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bartlomieju/lax](https://github.com/bartlomieju/lax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
