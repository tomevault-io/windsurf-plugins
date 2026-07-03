---
trigger: always_on
description: Inkwell is a WYSIWYG Markdown editor for React, built on Slate.js. The editor
---

# CLAUDE.md

## Project Overview

Inkwell is a WYSIWYG Markdown editor for React, built on Slate.js. The editor
content model is the Markdown source string. Markdown syntax is part of the
content; visual formatting is computed at render time and is never stored as a
separate rich-text model.

## Monorepo Structure

pnpm workspaces + Turborepo monorepo. Three packages, all in one workspace:

```
packages/
  inkwell/                     Library package: @railway/inkwell
    src/
      index.ts                 Public exports
      types.ts                 Public TypeScript types
      editor/inkwell-editor.tsx
      editor/slate/            Slate model, serialize/deserialize, features
      renderer/                Read-only renderer + renderer utilities
      plugins/                 Built-in plugins and tests
  inkwell-docs/                Astro Starlight docs + React demo island
```

## Commands

Run from the workspace root:

- `pnpm test` — Run all tests via turbo
- `pnpm dev` — Start docs dev server
- `pnpm build` — Build all packages
- `pnpm typecheck` — TypeScript type checking
- `pnpm lint` / `pnpm lint:fix` — Biome
- `pnpm changeset` — Add a changelog entry

Package-scoped validation used during API work:

- `pnpm --filter=@railway/inkwell typecheck`
- `pnpm --filter=@railway/inkwell test`
- `pnpm --filter=@railway/inkwell build`
- `pnpm --filter=inkwell-docs typecheck`
- `pnpm --filter=inkwell-docs build`

## Public API

`<InkwellEditor />` is the primary editor API.

```tsx
import { InkwellEditor } from "@railway/inkwell";
import { useState } from "react";

function App() {
  const [content, setContent] = useState("# Hello **world**");

  return <InkwellEditor content={content} onChange={setContent} />;
}
```

Use `ref={useRef<InkwellEditorHandle>(null)}` for imperative actions:

- `getState()`
- `focus({ at?: "start" | "end" })`
- `clear({ select?: "start" | "end" | "preserve" })`
- `setContent(content, { select?: "start" | "end" | "preserve" })`
- `insertContent(content)`

`setContent()` and `clear()` do not call `onChange`. `insertContent()` behaves
like a normal edit and flows through change handling.

The editor handle stays plugin-agnostic. Plugins that need an imperative
surface (e.g. click-to-attach for attachments) expose their own ref via
plugin options — see `AttachmentsHandle` and the `ref` option on
`createAttachmentsPlugin`.

The root package exports the component APIs, built-in plugin factories, renderer
utilities (`parseMarkdown(content, options)`, `htmlToMarkdown(html)`), and public
types. `RehypePluginConfig` accepts plugin tuples with rest options:
`[plugin, ...options]`. Do not export internal Slate helpers or shared plugin
primitives from the root API.

## Editor Rendering Model

Formatting is feature-based. The public prop is `features`.
All features are enabled by default:

- `headings` with optional `h1`–`h6` overrides
- `blockquotes`
- `codeBlocks`
- `images`

List markers (`-`, `*`, `+`, `1.`) stay plain text in the editor and are not
part of configurable editor features.

Inline Markdown styling is still implemented internally with Slate decoration
ranges. Public docs should call the configurable behavior “features.”

Inline marks include `bold`, `italic`, `strikethrough`, `inlineCode`, and
their `*Marker` counterparts, plus link marks: `link` (visible link text
— covers both the `[text]` label of `[text](url)` and the entire text of
bare URL autolinks), `linkUrl` (URL token inside `(url)`), and
`linkMarker` (`[`, `]`, `(`, `)` brackets). Links are always on; the
content model stays the markdown source — no link nodes, no separate
rich-text model. `[text](url)` and bare URLs (`https://...`, `www....`)
are matched after the inline-code pass and skip ranges already covered
by backticks or another link. Trailing punctuation (`.,;:!?`) is
trimmed off bare-URL matches so "see https://x.com." doesn't pull the
period into the link.

Paste-over-selection: when the clipboard payload trims to a single URL
and the editor selection is non-empty, `withMarkdown`'s `insertData`
override replaces the selection with `[selected](url)` instead of
dropping the bare URL. Paste with a collapsed selection (or empty
editor) follows the normal markdown-deserialize path — the URL ends up
as plain text and the autolink decoration picks it up. There is no
bubble-menu "insert link" button (the symmetric `wrapSelection`
primitive doesn't fit `[text](url)`); editing the markdown source IS
the link editing UX.

Use slot styling:

- `className` aliases `classNames.root`
- `classNames.root`, `classNames.editor`
- `styles.root`, `styles.editor`

Do not add a public top-level `style` prop.

The bundled stylesheet ships **no container-size opinion** (`min-height`,
`max-height`, `height`) on `.inkwell-editor`. Container sizing is a
consumer decision — fights between the library default and a chat
composer / panel embed / custom layout were not worth shipping. Demos and
docs set their own size via `styles.editor`.

Every visual-chrome default the stylesheet ships — colors, backgrounds,
borders, padding, typography — is wrapped in `:where()` so it carries
0,0,0 specificity. That covers `.inkwell-editor` and its inline marks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [railwayapp/inkwell](https://github.com/railwayapp/inkwell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
