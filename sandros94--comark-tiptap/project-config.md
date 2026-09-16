---
trigger: always_on
description: <!-- Keep this file updated as the project evolves. When making architectural changes, adding patterns, or discovering conventions, update the relevant sections. -->
---

<!-- Keep this file updated as the project evolves. When making architectural changes, adding patterns, or discovering conventions, update the relevant sections. -->

# comark-tiptap — Agent Guide

`comark-tiptap` is a [Comark](https://github.com/comarkdown/comark)-aware [Tiptap](https://tiptap.dev) kit that round-trips losslessly between Tiptap's ProseMirror schema, the Comark AST, and markdown. It ships as **one package with subpath entries**: `comark-tiptap` (framework-agnostic core), `comark-tiptap/vue` (Vue 3), `comark-tiptap/react` (React), and `comark-tiptap/internal` (binding plumbing, not semver-covered). Each framework and its Tiptap binding (`vue`/`@tiptap/vue-3`, `react`/`react-dom`/`@tiptap/react`) are **optional** peer deps.

## Core Principle — Ask First

**When in doubt, ask before acting.** Understanding the vision beats assuming. No wasted time in asking — this applies to every task.

### Q&A Sessions

For design decisions, ambiguity, or vision changes, run a structured Q&A before implementing:

- Each question: **2–4 labeled options** (A/B/C/D), 1–2 sentences each, with a marked preference. The answer can pick, mix, or override.
- No open-ended questions — propose options, even as best guesses.
- Number questions with a short kebab-case title for cross-reference.
- Prefer multiple focused rounds (2–5 questions each). Synthesize + confirm before implementing.

## Commands

- **Build:** `pnpm build` (obuild) — emits `dist/index.mjs` (core) + `dist/internal.mjs` + `dist/vue/index.mjs` + `dist/react/index.mjs`, each bundled with `.d.mts`.
- **Stub (dev):** `pnpm dev:prepare` — `obuild --stub` symlinks `dist/*` back to `src`, so playgrounds and `tsc` resolve the workspace `comark-tiptap` without a full build.
- **Test:** `pnpm test` (vitest). Single file: `pnpm vitest run test/serializer.test.ts`.
- **Nuxt integration:** `pnpm test:nuxt` (`@nuxt/test-utils/runtime`) — runs separately in CI; root Vitest excludes `playgrounds/**`.
- **Typecheck:** `pnpm typecheck` (`tsc --noEmit` — native TypeScript 7; the vue/nuxt playgrounds stay on TS 6 for vue-tsc).
- **Lint:** `pnpm lint` (`oxlint` type-aware + `oxfmt --check`). **Format:** `pnpm fmt`.
- **Playgrounds:** `pnpm dev:vue` / `pnpm dev:react` / `pnpm dev:nuxt`; `pnpm typecheck:playgrounds`.

## Architecture

Single package, subpath exports:

- `comark-tiptap` — `ComarkKit`, the serializer, per-node/mark specs, `defineComarkComponent`, utils. No framework code.
- `comark-tiptap/vue` — `<ComarkEditor>`, `useComarkEditor`, `defineComarkVueComponent`.
- `comark-tiptap/react` — `<ComarkEditor>`, `useComarkEditor`, `defineComarkReactComponent`.
- `comark-tiptap/internal` — the content-routing helpers the bindings share. Not part of the semver-supported public API.

Each framework binding imports the core **by package name** (`comark-tiptap`, self-referenced via `exports`), kept external at build time so the core is never re-bundled. Framework bindings mirror each other's surface, adapted to each framework's idioms (Vue `v-model` + modifiers; React controlled `value`/`onChange` + `contentType`).

### Source layout

```
src/
  index.ts              # core barrel
  internal.ts           # `comark-tiptap/internal` barrel — re-exports content.ts for the bindings
  kit.ts                # ComarkKit — assembles StarterKit + tables + image + picture + comark nodes + serializer
  serializer.ts         # ComarkSerializer extension + createSerializer (pure dispatcher) + PM↔Comark commands
  stream.ts             # stream session (progressive markdown) behind storage.comark.stream()
  content.ts            # @internal content-routing helpers shared by the bindings (applyContent/readByFlavor/isMarkdownDocumentLike/safeJson)
  attrs.ts              # ComarkAttrs — global `htmlAttrs` bag via addGlobalAttributes
  style.ts              # operational stylesheet (comment/template/component markers)
  types.ts              # NodeSpec / MarkSpec / ComarkHelpers + re-exported comark types
  extensions/           # comark-specific Tiptap nodes: code-block, comment, template, image (resolveSrc), picture, component (factory)
  specs/                # per-node/mark serialization specs (paragraph, heading, lists, table, marks, …) + exact.ts (1:1 tag factories) + comarkSpecs aggregate
  utils/                # attrs (split/merge/read the htmlAttrs bag), auto-unwrap (both paragraph-unwrap rules), html-attrs, srcset (candidate parsing), resolve-src (display URL + raw stash)
  vue/
    index.ts            # vue barrel
    comark-editor.ts    # <ComarkEditor> as a `defineComponent` (see "Build" below)
    use-comark-editor.ts# useComarkEditor composable
    define-component.ts # defineComarkVueComponent — wraps the core factory with VueNodeViewRenderer
    comark-editor.types.ts
  react/
    index.ts            # react barrel
    comark-editor.tsx   # controlled <ComarkEditor> (value/onChange) + BYO-editor branch
    use-comark-editor.ts# useComarkEditor hook (wraps @tiptap/react useEditor)
    define-component.ts # defineComarkReactComponent — wraps the core factory with ReactNodeViewRenderer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandros94/comark-tiptap](https://github.com/sandros94/comark-tiptap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
