---
trigger: always_on
description: <!--VITE PLUS START-->
---

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web

This project is using Vite+, a unified toolchain built on top of Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task. Vite+ wraps runtime management, package management, and frontend tooling in a single global CLI called `vp`. Vite+ is distinct from Vite, and it invokes Vite through `vp dev` and `vp build`. Run `vp help` to print a list of commands and `vp <command> --help` for information about a specific command.

Docs are local at `node_modules/vite-plus/docs` or online at https://viteplus.dev/guide/.

## Review Checklist

- [ ] Run `vp install` after pulling remote changes and before getting started.
- [ ] Run `vp check` and `vp run -r test` to format, lint, type check and test changes.
- [ ] Check if there are `vite.config.ts` tasks or `package.json` scripts necessary for validation, run via `vp run <script>`.
- [ ] If setup, runtime, or package-manager behavior looks wrong, run `vp env doctor` and include its output when asking for help.

<!--VITE PLUS END-->

# Project Notes for Agents

## Stack Snapshot

- Runtime/package management: Bun `1.3.14` through Vite+ `vp install`; Node.js
  `>=26.0.0`.
- Workspace tooling: Vite+ (`vp`) for dev, build, pack, check, test, preview,
  formatting, linting, type checking, task caching, and script orchestration.
- Language: TypeScript 6.x, ES modules, shared `tsconfig.*` files, and package
  builds through `vp pack`.
- Editor base: official `@codemirror/state`, `@codemirror/view`,
  `@codemirror/search`, and `@codemirror/lint` are allowed where they do not
  bring in Lezer-specific language behavior.
- Parser/runtime layer: `web-tree-sitter`, Tree-sitter grammar packages, WASM
  assets, highlight queries, and local CodeMirror-compatible wrappers. The root
  override currently resolves `web-tree-sitter` to `vendor/web-tree-sitter` for
  a local cursor range navigation binding patch.
- Product/runtime integrations: LiveMD uses KaTeX, Mermaid, and
  `beautiful-mermaid`; optional collaboration uses `loro-crdt` and
  `loro-codemirror`; `apps/local-md-workspace` uses React 19, shadcn/radix UI,
  i18next/react-i18next, the browser File System Access API, Dropbox OAuth
  PKCE, the OpenDAL browser WASM wrapper, and Grove shared-file relay clients;
  `apps/grove-relay` and `apps/collab-editor` use Cloudflare Workers, Durable
  Objects, WebSockets, Wrangler, and `@cloudflare/vite-plugin`.

## Repository Layout

- `packages/language`: Tree-sitter parser integration, syntax-tree wrappers,
  highlighting, indentation, folding, bracket matching, bidi isolation, and
  stream-parser compatibility.
- `packages/language-data`: `LanguageDescription` registry, Tree-sitter WASM
  asset loading, highlight-query loading, language metadata, and mixed-language
  parser wiring.
- `packages/commands`: Lezer-free editing commands, comments, history, and
  keymaps.
- `packages/autocomplete`: Completion state, sources, filtering, tooltips,
  snippets, word completion, and close brackets.
- `packages/codemirror`: `@codemirror-treesitter/basic-setup`, including
  `basicSetup`, `minimalSetup`, and the compatibility `EditorView` export.
- `packages/theme-palettes`: Shared concrete palettes reused by CodeMirror
  editor themes and LiveMD presentation themes.
- `packages/theme`: Shared semantic theme token contracts and CodeMirror
  editor/highlight extension factories for concrete theme packages.
- `packages/theme-gruvbox`: Local Gruvbox editor themes and highlight styles.
- `packages/theme-github`: Local GitHub editor themes and highlight styles.
- `packages/theme-catppuccin`: Local Catppuccin editor themes and highlight
  styles.
- `packages/merge`: Diff, split merge view, unified merge view, chunk helpers,
  and local highlighting for deletion widgets.
- `packages/lsp-client`: LSP transport, plugin, workspace mapping,
  diagnostics, completions, hover, formatting, rename, definition, references,
  and signature help.
- `packages/live-md`: Live Markdown editor runtime, web component, registration
  entry, CSS export, and fixtures.
- `packages/live-md-theme`: Shared LiveMD presentation token contract and host
  variable helpers.
- `packages/live-md-theme-gruvbox`: Gruvbox LiveMD presentation themes.
- `packages/live-md-theme-github`: GitHub Light LiveMD presentation theme.
- `packages/live-md-theme-catppuccin`: Catppuccin LiveMD presentation themes.
- `packages/live-md-loro`: Optional Loro collaboration bindings for LiveMD.
- `packages/opendal-wasm-browser`: Experimental browser WASM wrapper for
  OpenDAL-backed cloud workspace storage.
- `vendor/web-tree-sitter`: Vendored `web-tree-sitter` runtime package used by
  the root override while the local cursor range navigation binding patch is
  needed.
- `apps/basic-editor`: Minimal `<live-md-editor>` smoke app.
- `apps/local-md-workspace`: Grove React shadcn/radix local-first Markdown
  workspace using LiveMD, the browser File System Access API, optional Dropbox
  storage through OpenDAL WASM, local image asset handling, and optional Grove
  shared-file collaboration through `apps/grove-relay`.
- `apps/grove-relay`: Grove shared-file relay Worker with Durable Object
  persistence, WebSocket Loro sync, share lifecycle APIs, and Wrangler deploy
  config.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eric-Song-Nop/codemirror-treesitter](https://github.com/Eric-Song-Nop/codemirror-treesitter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
