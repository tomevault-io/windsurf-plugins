---
trigger: always_on
description: > **For agentic workers:** Read this file fully before touching any code. It contains the codebase map, conventions, and workflow required to work here safely.
---

# markdown-docx — Agent Entrypoint

> **For agentic workers:** Read this file fully before touching any code. It contains the codebase map, conventions, and workflow required to work here safely.

---

## What This Project Is

`markdown-docx` converts Markdown to `.docx` (Microsoft Word) format. It runs in **both Node.js and browser** environments. It is published as an npm package with a CLI tool.

**Stack:** TypeScript · [docx](https://github.com/dolanmiu/docx) · [marked](https://marked.js.org/) · [KaTeX](https://katex.org/) · [tsdown](https://github.com/rolldown/tsdown) (build) · [vitest](https://vitest.dev/) (tests) · pnpm

---

## Repository Layout

```
src/                    Library source
  index.ts              Public entry (Node.js) — default export markdownDocx(), re-exports all types
  entry.ts              Browser entry
  entry-node.ts         Node.js-specific entry (includes image adapter)
  MarkdownDocx.ts       Core class — owns options, styles, footnotes, render dispatch
  tokenize.ts           Wraps marked tokenizer; attaches custom extensions
  utils.ts              Pure helpers (heading level map, image extension, resolvePageMargins)
  types/                All TypeScript types (pure type files, no runtime code)
    index.ts            Barrel — re-exports everything from sub-files
    theme.ts            IMarkdownTheme — colors, sizes, margins, collapseEmptyLines
    options.ts          MarkdownDocxOptions — full API options interface
    image.ts            MarkdownImageType, MarkdownImageItem, MarkdownImageAdapter
    token.ts            IBlockToken, IInlineToken, IParagraphToken (union types)
    attr.ts             ITextAttr, IBlockAttr (render attribute objects)
    style.ts            IMarkdownToken, IMarkdownStyle, IMarkdownRenderFunction, Writeable
  adapters/
    browser.ts          Canvas-based image adapter for browsers
    nodejs.ts           fetch/sharp-based image adapter for Node.js
  extensions/
    index.ts            Registers marked extensions; re-exports extension types
    footnote.ts         Footnote extension (block + inline)
    latex.ts            KaTeX inline/block extension
    mathml-to-docx.ts   Converts MathML XML → docx Math children
    types.ts            Extension-specific token types
  renders/
    index.ts            Re-exports renderBlocks and renderTokens
    render-blocks.ts    Block-level token → docx FileChild (paragraph, table, list…)
    render-tokens.ts    Inline token dispatch
    render-paragraph.ts Paragraph construction
    render-text.ts      Run (text span) construction
    render-list.ts      Ordered/unordered/task lists
    render-table.ts     Table rendering
    render-image.ts     Image rendering
    render-checkbox.ts  Task list checkbox
  styles/
    index.ts            Re-exports; builds default styles object
    themes.ts           defaultTheme constant (IMarkdownTheme values)
    styles.ts           createDocumentStyle(), createDefaultStyle()
    markdown.ts         createMarkdownStyle() — per-token paragraph/run styles
    classes.ts          CSS class name constants (MdParagraph, MdHeading1…)
    numbering.ts        List numbering configuration

bin/
  markdown-docx.mjs     CLI entry (Commander.js); --input --output --theme --config
  help-texts.mjs        CLI help content for `markdown-docx help theme|config`

tests/                  Integration tests (vitest)
  index.test.ts         Full-document round-trip tests
  margin.test.ts        Unit tests for resolvePageMargins()
  theme-styles.test.ts  Unit tests for createDefaultStyle()
  theme-CLI.test.ts     CLI flag tests (execSync)
  test-math.test.ts     Math/KaTeX rendering tests
  *.md                  Sample markdown inputs

src/**/*.test.ts        Unit tests co-located with source
  src/index.test.ts     Per-feature render tests (snapshots)
  src/tokenize.test.ts  Tokenizer unit tests
  src/__snapshots__/    Vitest snapshot files (auto-generated)

examples/
  sample-config.json    Full config reference with all defaults documented

docs/superpowers/
  plans/                Implementation plans (YYYY-MM-DD-<feature>.md)
  specs/                Feature specs
```

---

## Architecture Mental Model

```
markdownDocx(markdown, options)
  └─ new MarkdownDocx(markdown, options)
       ├─ tokenize()          → IBlockToken[]  (marked + extensions)
       ├─ downloadImageList() → populates _imageStore
       └─ toBlocks()          → FileChild[]
            └─ renderBlocks() → per-token → renderParagraph / renderList / renderTable / …
                                  └─ renderTokens() → ParagraphChild[] (inline spans)

new Document({ styles, numbering, sections: [{ properties, children }] })
```

The `MarkdownDocx` instance is threaded through all render functions as `render`. Access options via `render.options`, theme via `render.options.theme`.

---

## Key Conventions

### Types
- **All types live in `src/types/`**. Never define types in other files.
- `index.ts` is a pure barrel — only `export type { … } from './file'` lines.
- Add new types to the most semantically appropriate file (`theme.ts`, `image.ts`, `attr.ts`, `token.ts`, `style.ts`, `options.ts`).

### Styles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vace/markdown-docx](https://github.com/vace/markdown-docx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
