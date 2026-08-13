---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
npm run build          # tsup → patch-modern-regex → copy-miniprogram-dist → copy-component-assets
npm test               # vitest run --coverage  (one-shot, not watch)
npm run test:ci        # same + JSON reporter → test-results.json (CI gate consumes this)
npm run lint           # eslint src --ext .ts
npm run bench          # tinybench → benchmark/results.json
npm run bench:check    # bench + fail if any x-markdown-mini scenario regressed >10% vs baseline.json
npm run bench:update   # bench + promote results.json → baseline.json (commit alongside perf change)
npm run check:bundle   # size budgets + ES2018 syntax (es-check) + no named-group regex leak — needs prior build
npm run check:test-rate -- --min 95  # gate on pass rate from test-results.json
npm run docs           # build + dumi dev server (docs-site/), live preview of mini-program shells
```

Single test: `npx vitest run src/__tests__/tokensToWechat.test.ts` (path) or `npx vitest run -t "<a href>"` (name filter).

`npm test` re-runs the full suite; vitest watch mode is intentionally not the default — CI and local dev both want the one-shot.

## Public API surface

Three entry points, all backed by a shared `defaultInstance = new XMarkdownMini()` singleton in `src/index.ts`:

- `parse(content)` → marked `Token[]`. Pure lex, no platform mapping.
- `render(content)` / `render(props)` → marked `Token[]`. Same as parse, but the props form supports `streaming`/`onPatch` callbacks for streaming token consumers who do their own rendering.
- `renderNodes(props)` → `MiniNode[]` for mini-program components. Picks a platform renderer based on `props.platform` (default `'auto'`).

For concurrent streams, **construct `new XMarkdownMini(opts)` per view** — the singleton's streaming state is shared across callers. The bundled mini-program components do this in `attached`/`didMount` and `reset()` in `detached`/`didUnmount`.

`XMarkdownMiniOptions`:
- `escapeText` (default `true`) — HTML-entity escape text node values. The default is `true` because the `MiniNode` shape is rich-text-nodes-compatible by lineage (and `<rich-text>` would decode entities); the bundled `MiniNodeRenderer` components pass `false` because their `<text>{{value}}</text>` does not decode entities.
- `streamingFixup` (default `'remend'`) — tail auto-completion of unfinished markdown during streaming (`**bo` → `**bo**`, etc.). `false` disables it; a function replaces it. Only the streaming paths use it; one-shot parse/render never call fixup.
- `gfm` (default `true`) — GFM tables, strikethrough, autolinks. Per-call `renderNodes(props)` / `render(props)` accept a `gfm?` override.
- `breaks` (default `false`) — soft line breaks become `<br>`. Per-call `renderNodes(props)` / `render(props)` accept a `breaks?` override.
- `extensions: (XMarkdownExtension | MarkedExtension)[]` — forwarded to `new Marked(...)`, instance-scoped (never mutates the global marked singleton). Extensions are baked in at construction and cannot vary per call.
- `components: string[]` — whitelist of literal custom-component tags that get an auto-synthesized inline tokenizer (`<ant-button>X</ant-button>` → MiniNode named `ant-button`). User-registered extensions with the same name win over the synth.

## Architecture: direct-to-platform transformer

```
markdown string → marked.lexer → Token[] → tokensToWechat / tokensToAlipay → MiniNode[]
```

`renderNodes` resolves the target platform (`resolvePlatform`), looks up a `PlatformRenderer` from `RENDERERS` in `src/platforms/index.ts`, and calls `renderer.renderTokens(tokens, ctx)`. There is **no intermediate IR layer, no separate adapter step, no platform-agnostic transformer**. Platform quirks (wechat's `<a data-href>` rewrite, alipay's https-only images, alipay dropping `<ol start>`) are baked directly into the per-platform transformer. The output `MiniNode` tree (`{ name, attrs, children, animate }`, hast-shaped — rich-text-nodes-compatible by lineage) is rendered by the in-repo `MiniNodeRenderer` component, which dispatches on `node.name` to native primitives (`<text>` / `<view>` / `<image>` / `<scroll-view>`). We render the tree ourselves rather than feeding it to native `<rich-text>`: `<rich-text>` is an HTML-rich-text renderer that would re-impose a tag/attr whitelist, strip events, and block per-node animation — pointless when the tree is already our own structured data.

See `docs/experiments/2026-05-pipeline-architecture.md` for the empirical A/B/C comparison that drove this design (C beat A by ~17% throughput, eliminated UnifiedNode intermediate and capability-matrix adapter).

- `src/platforms/wechat/tokensToWechat.ts` / `src/platforms/alipay/tokensToAlipay.ts` — independent transformers, ~95% structurally identical, ~12 lines of platform-specific divergence. `marked` is a real dependency bundled via tsup `noExternal`.
- **Platform list**: `Platform` is `'wechat' | 'alipay'` (declared in `src/platforms/types.ts`). Default fallback when runtime detection fails is `'alipay'`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ant-design/x-markdown-mini](https://github.com/ant-design/x-markdown-mini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
