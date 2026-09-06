---
trigger: always_on
description: **Prismaliser** is a fully client-side webapp (Vite SPA) that visualises
---

# Repository Guidelines

## Project Overview

**Prismaliser** is a fully client-side webapp (Vite SPA) that visualises
[Prisma](https://prisma.io) schemas as ER diagrams. Users paste a schema into a
Monaco editor; the app parses it **entirely in the browser** (via Prisma's
schema WASM module) and renders models, enums, and relations (1-1, 1-n, m-n) as
an interactive React Flow graph. The build output is plain static files
(`dist/`) — no server component at all. It is self-hostable (Docker image
published to GHCR); a hosted version lives at
[prismaliser.app](https://prismaliser.app).

## Architecture & Data Flow

The app is a single page (`src/App.tsx`) — editor on one side, graph on the
other.

```
Monaco editor (EditorView)
  → schema text, debounced 1s (react-use useDebounce)
  → util/prisma.ts  (getDMMF / formatSchema — client-side WASM wrappers)
      → @prisma/prisma-schema-wasm  (patched; instantiated from
         /prisma_schema_build_bg.wasm fetched out of public/)
  → DMMF.Datamodel  (or PrismaSchemaError → Monaco markers)
  → components/FlowView.tsx
      → util/prismaToFlow.ts  (DMMF → React Flow nodes/edges)
      → util/layout.ts        (elkjs layered layout, DOWN)
      → ReactFlow canvas      (ModelNode / EnumNode / RelationEdge)
```

Key points:

- **Parsing/formatting is client-side WASM.** `@prisma/prisma-schema-wasm` is
  the Rust `prisma-fmt` engine compiled to WASM — pure JS+WASM, no native code.
  The published package is a Node-only build (loads the binary via
  `fs`/`__dirname`), so the repo carries a **Yarn patch** (`.yarn/patches/`,
  wired via the `patch:` protocol in `package.json`) that replaces the
  self-instantiating fs tail with an exported `__init(wasmBytes)`.
- **The wasm binary is vendored at `public/prisma_schema_build_bg.wasm`** (2.9
  MB) and fetched once, eagerly, by `src/util/prisma.ts`. ⚠️ It is a copy of
  `node_modules/@prisma/prisma-schema-wasm/src/prisma_schema_build_bg.wasm` —
  re-copy it when updating the pinned package version, and regenerate the Yarn
  patch.
- **Error contract:** the wasm throws `Error`s whose message is a JSON
  `{ error_code, message }` blob. `util/prisma.ts` unwraps it and throws
  `PrismaSchemaError` (carrying `SchemaError[]` for Monaco markers) when the
  message contains `error: ` diagnostics, otherwise a plain `Error`.
  `parseDMMFError` (`util/index.ts`) does the line-number extraction and works
  unchanged.
- **Monaco is bundled, not CDN-loaded** (`src/monaco.ts`):
  `loader.config({ monaco })` with a Vite `?worker` import, so the app works
  fully offline. Consequently `window.monaco` does **not** exist — drive the
  editor in tests/tools via share links (`?code=`) or DOM, not the monaco
  global.
- **No global state store** (no zustand/redux/context). State is local
  `useState` in `src/App.tsx` and `components/FlowView.tsx`; the schema persists
  via `useLocalStorage("prismaliser.text")` from `react-use`.
- **m-n relations create implicit virtual tables** in `prismaToFlow.ts` (IDs
  like `_${relationName}`, columns `A`/`B`) — the graph intentionally differs
  from the raw schema.
- **Layout is not automatic.** On `dmmf` change, nodes regenerate at `{0,0}` (or
  previous positions) until the user clicks "Disperse nodes", which runs elkjs.
- **Handle ID coupling:** `ModelNode`/`RelationEdge` must agree with the handle
  ID strings generated in `prismaToFlow.ts` (`relationEdgeSourceHandleId`,
  `relationEdgeTargetHandleId`, `enumEdgeTargetHandleId`). Change both sides
  together.
- **Share links:** schema is URL-safe-base64-encoded into `?code=`
  (`toUrlSafeB64`/`fromUrlSafeB64` in `util/index.ts`). There is no client-side
  router; query params only, so static hosting needs no rewrite rules.

## Key Directories

| Path              | Purpose                                                                                                                                                                                                                               |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/`            | All application code. `main.tsx` (entry), `App.tsx` (the whole page), `monaco.ts` (editor setup).                                                                                                                                     |
| `src/components/` | React components: `FlowView`, `ModelNode`, `EnumNode`, `RelationEdge`, `EditorView`, `Layout`, …                                                                                                                                      |
| `src/util/`       | Core logic: `prisma.ts` (client-side parse/format), `prismaToFlow.ts` (DMMF→graph), `layout.ts` (elkjs), `prisma-language.ts` (Monarch grammar), `types.ts` (shared contracts), `index.ts` (helpers). Tests colocated as `*.test.ts`. |
| `src/assets/`     | `style/global.css` — Tailwind entrypoint + custom `.button`/`.focusable` utilities.                                                                                                                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ovyerus/prismaliser](https://github.com/Ovyerus/prismaliser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
