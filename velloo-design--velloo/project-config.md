---
trigger: always_on
description: This file orients you when you're modifying **the Velloo repo itself**. For guidance when you're touching a *Velloo design folder* via MCP, see the MCP server's `initialize` instructions; those concerns are separate from the substrate.
---

# CLAUDE.md — repo-level guide for AI agents

This file orients you when you're modifying **the Velloo repo itself**. For guidance when you're touching a *Velloo design folder* via MCP, see the MCP server's `initialize` instructions; those concerns are separate from the substrate.

## Mental model

Velloo is a local, code-shaped design canvas for solo devs. The repo is a Bun-workspaces monorepo split into sixteen packages with one-way dependencies:

```
schema → result → protocol → provider → helpers → shadcn-snapshot → provider-none, provider-mui, provider-shadcn-upstream, provider-antd, provider-chakra → renderer → codegen → server → canvas → cli
```

`protocol` is the wire contract (mutation arguments, typed errors, watch
events, the cloud surface, the publish bundle) — imported by `server`,
`canvas`, `cli`, and by velloo-cloud.

The cleanest packages (`schema`, `result`, `provider`) have no internal runtime deps. Everything else builds on them. **Do not introduce cycles** — every cross-package import must respect this order.

**Velloo is framework-native.** A folder *targets a framework*: shadcn (Tailwind `className`), MUI (`sx` + emotion, real `@mui/material`), or no-framework (bare primitives). The `ComponentProvider` grew into a **`FrameworkAdapter`** (`packages/provider/src/adapter.ts`) owning the style channel, render pass, codegen module, theme projection, catalog/install, and the canvas bundle. So: styling is **not** universally Tailwind (`set_style` routes through `styleChannelOf(provider, folderCss)`); codegen emits the screen framework's native idiom (a `CodegenTarget`); and the canvas bundle can mix exact host files, canvas-safe adaptations, and fallbacks in one screen. When you touch styling, rendering, codegen, or the inspector, resolve the **per-screen adapter** (`providerForScreen`) and its `StyleChannel` — never assume shadcn/Tailwind.

**Components also come from the app itself.** The repository catalog (`packages/server/src/repo/`) lays the components the host app actually renders — a package it depends on (Mantine, a private design system) or its own `components/` — over whatever provider the folder uses. A node that is one carries `$repo` identity (`{ importPath, exportName, member?, app?, proxy? }`); `$ref` is then only its JSX name, so never look a `$repo` node up in a provider manifest by `$ref` (Mantine's `Button` is not the provider's `Button`). Such nodes render for real in the canvas client mount inside the design's preview entry (`preview.tsx`, or a framework recipe in `repo/recipes/`), fall back per component to a proxy snippet or labelled frame, style only through the props they declare, and emit with their exact imports (`repoImports`). A library without an adapter gets a recipe, not a provider.

**The CSS framework is its own axis, independent of the UI library.** `config.styling.framework` (`"tailwind"` | `"none"`, detected by `init`) picks the channel for a provider that supports more than one: shadcn is always Tailwind and MUI always `sx` (intrinsic, single-channel), but **no-framework** pairs with Tailwind *or* `none` (inline `style` objects, no JIT — themed via the CSS vars `themeToCss` injects). So a `none/none` folder renders inline-styled primitives (`provider.registryForChannel("style")`), the inspector edits an inline `style` object, and `emit_code` produces Tailwind-free `style={{…}}` on plain HTML. `styleChannelOf(provider, folderCss)` is the one resolver; the provider's `styleChannels` allowed-set bounds what `folderCss` can pick, and `resolveProviders` rejects an incoherent pair (shadcn + none).

### Package responsibilities

- **`@velloo/schema`** — Zod schemas + TS types for everything on disk in a design folder (Screen, Board, Frame, Snippet, Theme, Config, Node, Annotation, CanvasNote). Plus pure utilities like `collectIds` / `findDuplicateIds`. **No I/O. No framework imports.** This package is the contract between every other package — keep it minimal.
- **`@velloo/result`** — `Result<T, E>` helpers (`ok`, `err`, `unwrap`, the `DoAsync` generator monad). Used throughout for typed errors instead of throwing.
- **`@velloo/provider`** — The `ComponentProvider` interface every library entry implements (shadcn, no-lib, MUI, future host-scan). Owns the `Manifest` / `ComponentDescriptor` / `PropDescriptor` types. No runtime code beyond the loader plumbing — concrete providers live in their own packages.
- **`@velloo/helpers`** — The framework-neutral velloo helper components (Box, Heading, Text, Icon, Image, SVG, Layer, Divider, Gradient, Placeholder) plus their canonical `ComponentDescriptor` set (`HELPER_DESCRIPTORS` / `helperDescriptors(ids)`), the `helpersRegistry(ids)` subset selector every provider reuses, the codegen lowering class tables (`src/lowering.ts`), and `helpersComponentsDir` (the Tailwind JIT scans it so helper default classes always compile). Sits between `provider` and the concrete providers; no library idioms.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [velloo-design/velloo](https://github.com/velloo-design/velloo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
