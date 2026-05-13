---
trigger: always_on
description: A Typora-style WYSIWYG Markdown editor built on ProseMirror.
---

# typora-web

A Typora-style WYSIWYG Markdown editor built on ProseMirror.

## State model

```
md text  ───────┐                                          (IO boundary; on-disk format)
               parse / serialize (doc must round-trip losslessly)
PM doc + selection  ───── runtime authority
  │   textblock text keeps source delim chars (`*`, `**`, `` ` ``, `~~`, `[`/`](href)`)
  │   inline marks = derived each transaction by `normalize` (method B)
  ↓
normalize delim ranges  — inline decorations (`syntax-hint` / `syntax-hidden`)
view (DOM, built by PM EditorView)
```

- **Runtime authority** is `EditorState = { doc, selection, storedMarks, plugins }`. Transactions and decoration logic live at the PM doc layer.
- **md is not reactive**: it only appears at load / save / "show source" boundaries.
- **Method B is the only path** for inline features (em / strong / code / strike / link). The textblock's textContent contains the source delim chars verbatim; the corresponding mark is derived in `normalize.ts`'s `appendTransaction` by running `parseInline`. Flip side: those delim chars round-trip through the serializer raw (not `\*` escaped), because they *are* the md source.
- **Lossless round-trip** is an invariant over nodes/marks/attrs only. `selection`, `history`, decorations, IME state are ephemeral.
- See `~/.claude/projects/-Users-yanzhen-fiddle-typora-web/memory/project_state_model.md` for the original rationale.

## File map

### Top-level layout

The repo splits four ways. The dependency rule is one-way: `src/`
never imports from `tests/` / `specs/` / `website/`. That's what
keeps lib mode's bundle free of test fixtures and harness UI.

```
src/         editor lib (lib mode entry: src/lib.ts)
specs/       test/spec data — cases, renderCases, event DSL, pretty
tests/       *.test.ts + runFeatureCases driver
website/     harness UI — main.ts, index.html, style.css
```

`tsconfig.json` includes all four; vite's website build uses
`root: "website"`; lib build is configured via `vite.lib.config.ts`.

### Core (`src/`)

| File | Responsibility |
|---|---|
| `lib.ts` | Public API. Re-exports `defaultPlugins`, `createState`, `schema`, `parse`, `serialize`. Lib bundle's import root — must not transitively reach specs/tests/website. |
| `schema.ts` | PM schema. Merges `coreNodes` (doc/paragraph/text/hard_break/heading/blockquote/code_block/lists) with `collectNodes()` / `collectMarks()` from features. No core marks — every mark belongs to a feature. |
| `parser.ts` | md → PM doc. `ParserState` is exported for features (with `topMark(type)` so close-handlers can read attrs before closing). Core block / text / softbreak / hardbreak tokens handled inline; all other tokens dispatched through `collectParserTokens()`. Registered `mdItPlugins` run on the singleton markdown-it instance. |
| `serializer.ts` | PM doc → md. Mark delimiters come from `collectMarkDelims()`; each inline feature's `extRanges(parent)` marks the chars that must be emitted raw (no backslash escape) so method-B delim chars survive round-trip. |
| `inline-parse.ts` | Method-B inline parser. Utilities: `scanRuns`, `scanFixedDelim`, `markConsumed`, `markExtRanges`. Orchestration: `parseInline(text)` runs every inline feature's `scan` in ascending `priority`, sharing a `consumed` bitmap. `InlineSpan.attrs` lets attr-bearing marks (link's `{href, title}`) flow through normalize. |
| `normalize.ts` | The authoritative "text → marks" step. `appendTransaction` walks every textblock, runs `parseInline`, and syncs em / strong / code / strike / link marks. Plugin state exposes `delim` ranges for decorations. Attr-bearing marks are always re-applied (bitmap diff can't detect href/title changes). |
| `decorations.ts` | `syntaxHintsPlugin` — reads `getDelims(state)` from normalize and wraps each delim range in an inline `Decoration` with class `syntax-hint` (cursor in the surrounding span) or `syntax-hidden` (outside). Single path; the old widget path is gone. |
| `cursor-render.ts` | `cursorRenderPlugin()` — paints the selection as a widget. Empty selection → `<span class="play-caret">`, non-empty → `selection-marker` on both ends. Since gray delims are now inline decorations on real chars, no dynamic `side` juggling. |
| `input-rules.ts` | Thin shell: `inputRules({ rules: collectInputRules(schema) })` + `spaceBreaksStoredMarks`. Under method B, no inline feature registers input rules — normalize handles everything. Input rules remain available for block-shaped syntaxes that aren't delim-pair based. |
| `editor.ts` | `defaultPlugins()`: history / keymap / input-rules / space-breaks / **normalizeInlinePlugin** / syntaxHints / cursorRender / baseKeymap. Same stack in live editor and test pretty. |

### Features (`src/features/`)

| File | Owns |
|---|---|
| `_types.ts` | `FeatureSpec`, `InlineFeatureSpec`, `TokenHandler`. (Test-data types — `Case`, `Checkpoint`, `RenderCase`, `FeatureSpecs` — live in `specs/_types.ts`.) |
| `index.ts` | `ALL_FEATURES` registry plus `collectX()` helpers that core modules read. Adding a feature = one import + one array entry. |
| `emphasis.ts` | em + strong (priority 2). Merged because they share one `*`/`_` runs scanner (strong wins when both ends have ≥ 2 chars). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yuyz0112/typora-web](https://github.com/Yuyz0112/typora-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
