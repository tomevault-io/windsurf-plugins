---
trigger: always_on
description: Verter is a Vue compiler and Language Server Protocol (LSP) implementation. It converts Vue Single File Components (SFCs) to valid TSX (leveraging TypeScript for type checking) and compiles templates to optimized render functions. Unlike Volar, Verter generates actual valid TSX code rather than virtual files.
---

# Verter

Verter is a Vue compiler and Language Server Protocol (LSP) implementation. It converts Vue Single File Components (SFCs) to valid TSX (leveraging TypeScript for type checking) and compiles templates to optimized render functions. Unlike Volar, Verter generates actual valid TSX code rather than virtual files.

The project is a hybrid Rust + TypeScript monorepo: Rust crates handle template compilation (exposed via NAPI-RS native bindings and wasm-bindgen WASM) and the LSP server (`verter_lsp` binary, communicates over stdio), while TypeScript packages handle the SFC-to-TSX transformation and IDE integration.

## Architecture

### Shared Optimized Codebase (CRITICAL)

Verter is one shared optimized codebase, not separate semantic implementations per consumer.

- Improvements should land in the lowest reusable owner crate that can correctly serve all consumers.
- `verter_host` and shared workspace/VFS integration are the authority for host-backed loading, invalidation, dependency tracking, and cache reuse.
- `verter_analysis` and `verter_core` own reusable semantics and type-resolution logic.
- `verter_ffi` owns shared boundary DTOs between native bindings and WASM bindings.
- Consumer packages such as `@verter/component-meta`, the LSP, MCP, unplugin, and playground should consume the shared substrate rather than carrying their own semantic forks.

Architectural consequence:

- A performance or correctness fix discovered in one surface should be implemented in the shared owner layer whenever that behavior is reusable.
- Consumer-local wrappers should stay thin and should not bypass shared parsing, analysis, resolution, or cache ownership.

### Macro Type Traversal Rule (CRITICAL)

When resolving cross-file macro types (`defineProps<T>()`, `defineEmits<T>()`, component-meta deep expansion, etc.), only follow the import graph reachable from the requested type's declaration graph.

- There is one shared cross-file type resolver for host-backed component-meta and analysis work. Do not add consumer-specific resolver forks.
- That resolver has exactly two modes:
  - `Type`: resolve the requested symbol identity and canonical source location only. Do not expand the shape.
  - `Expanded`: resolve the same symbol through the same traversal, then materialize the expanded shape / expanded text.
- Component-meta uses the shared resolver in `Expanded` mode for every macro-facing surface. This includes all script-setup macros and all Options API surfaces that contribute metadata, such as props, data, computed, emits, slots, and expose-like members.
- Do not walk unrelated imports from the same file.
- Do not treat plain imports as implicit exports.
- Keep direct re-exports (`export { X } from`, `export * from`) as an explicit separate path.
- Parsing a `.ts`/`.js`/declaration file for type resolution must cache discovered symbol name → canonical location mappings.
- Re-exported names and barrel hops must also be cached once discovered. If traversal follows `export * from './foo'`, cache that result so later lookups do not rescan the same barrel chain.

If a file imports 20 modules but the requested macro type only references `AvatarProps` and `IconProps`, external resolution must only traverse those reachable dependencies.

**TS-first resolution priority:** TypeScript types always take priority over JavaScript files when resolving ambiguous dependency candidates. Verter is a type-strict compiler that relies on TS typing for correctness. JS files should only be used as a last resort when no TS type definition is available. When `DependencyResolution.possible_canonical_ids` contains multiple candidates, use `effective_target()` which selects the single highest-priority candidate: `.d.ts` > `.d.cts` > `.d.mts` > `.ts` > `.tsx` > `.js` > `.jsx` > `.cjs` > `.mjs`. Do not try remaining candidates if the selected one lacks the needed type — treat as not found.

### Canonical Dependency Cache Rule (CRITICAL)

Host-backed type/import resolution must treat the canonical file ID as the cache identity. The cache contract is:

- Load a dependency source at most once per canonical ID per workspace content generation. Parse it immediately and cache the raw source, parsed/OXC snapshot, and any reusable eval/build state right away.
- When the host materializes an imported dependency on a cold miss, derive the AST-backed bundle from that single parse and cache it together: file snapshot, eval env, external-type analysis, symbol/export lookup tables, and any other reusable per-file analysis. Do not let later resolver stages trigger a second parse of the same canonical file just to build another artifact.
- Cache named declarations from that parsed file by name, not just exported entrypoints. Internal named types/interfaces/aliases still matter because exported declarations in the same file may depend on them later.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pikax/verter](https://github.com/pikax/verter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
