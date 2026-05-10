---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Type-check then bundle the extension
npm run build

# Watch mode (dev builds, no type-check)
npm run build:watch

# UI preview in browser — no extension APIs needed
npm run dev          # then open http://127.0.0.1:5174/panel.html

# Tests
npm test             # run once (vitest)
npm run test:watch   # watch mode

# Run a single test file (tests are colocated: src/shared/*.test.ts)
npx vitest run src/shared/query.test.ts
```

After `npm run build`, load `dist/` as an unpacked extension in Chrome DevTools mode.

## Repo layout

- `src/` — the Chrome extension source (see Architecture below).
- `plan/` — numbered, durable feature plans (`00-overview.md`..`20-*.md`). Consult before starting a new feature — scope, constraints, and rationale usually already live there.
- `docs/` — supporting docs (`BUGS.md`, `DESIGN.md`, `PROJECT_BOARD.md`, Chrome Web Store copy, design prototypes under `docs/prototypes/`).
- `landing-site/` — **separate** Next.js marketing site (its own `package.json`, `CLAUDE.md`, build). Not part of the extension build. Run its own scripts from inside that directory; never import from it or let it affect extension bundling.
- `devtools.html`, `panel.html` — Vite build entry points at repo root, referenced from `vite.config.ts`. Don't move them.

## Architecture

This is a **Manifest V3 Chrome DevTools extension** packaged with Vite + `@crxjs/vite-plugin`. There is no separate dev server for extension development — you must run `npm run build` and reload the extension.

### Three-process model

```
Chrome DevTools panel (panel.html / src/panel/main.tsx)
    ↕ chrome.runtime port (PanelMessage / PanelReply)
Service worker (src/background/index.ts)
    ↕ chrome.scripting.executeScript (world: "MAIN")
Inspected page (injected inline — no content script file)
```

**Panel** (`src/panel/main.tsx`) — the entire UI. A single React root with a `useStorageRpc()` hook that sends typed `StorageRequest` messages over a persistent port and awaits `StorageResponse` replies. All state lives here; there is no external store.

**Background service worker** (`src/background/index.ts`) — receives RPC requests and routes them to the inspected page via `chrome.scripting.executeScript`. Handles `discover` specially: it scans all scriptable frames in parallel and merges their IndexedDB databases (each frame origin has its own partition). `executeStorageRequest` is a self-contained function injected into the page's MAIN world — it must not reference any module-scope variables.

**Injected logic** — the entire IDB cursor loop, MongoDB-style filter matcher, and KV read/write all run inside `executeStorageRequest` in `background/index.ts`. Serialization of non-JSON types (BigInt, Date, RegExp, Map, Set, ArrayBuffer, Blob, circular references) happens here before values cross the message boundary.

### Shared types (`src/shared/`)

| File | Purpose |
|---|---|
| `types.ts` | All cross-process types: `StorageRequest`, `StorageResponse`, `IndexedDbRecord`, `SerializedCell`, `NoSqlQuery`, etc. |
| `query.ts` | `parseMongoQuery()` — validates and normalises the JSON query string into `NoSqlQuery`. Also has `getPathValue()` for dotted-path access. |
| `filters.ts` | Client-side filter bar: `FilterState`, `FilterRule`, `applyFilters()`, `activeRuleCount()`. Operators applied in the panel, not in the injected script. |
| `indexed.ts` | `keyStrategy()` — infers whether a store uses auto, inline, or out-of-line keys from `IndexedDbStoreInfo`. Used when constructing draft rows. |
| `schemaInfer.ts` | `inferSchema(rows)` — samples up to 500 rows and returns `InferredColumn[]` (name, type, nullable, coverage). Used to power autocomplete and the Structure view. |
| `schemaExport.ts` | `toTypeScript()` / `toDexieSchema()` — exports inferred schema as code. |
| `persisted.ts` | Extension-owned IndexedDB (`idxbeaver` DB) for query history and saved queries. History is auto-trimmed to 100 entries per origin. |
| `prefs.ts` | `Prefs` type and `chrome.storage.local` helpers (`getPrefs`, `setPrefs`, `watchPrefs`). Key: `prefs.v1`. |
| `serialize.ts` | Utilities for round-tripping `SerializedCell` values. |
| `wire.ts` | Versioned wire format (`WIRE_VERSION = 2`) with `$t`-tagged envelopes for round-tripping arbitrary JS values (BigInt, Date, RegExp, Map, Set, TypedArrays, etc.). All persisted artefacts (snapshots, archive manifests) are stamped with this version — bump it only with a migration. |
| `rpcIds.ts` | `IDEMPOTENT_TYPES` set and `isIdempotent()`, plus pending-request metadata in `chrome.storage.session` so requests can be safely re-dispatched after a service-worker restart. |
| `undo.ts` | `UndoStack` — capped undo/redo stacks for `putRecord` grid-edit commands (default cap 100). |
| `export.ts` / `import.ts` | Multi-format readers/writers (JSON, NDJSON, CSV, SQL, ZIP) for the import/export dialogs. `import.ts#detectFormat(file)` inspects filename and MIME. |

### Panel components (`src/panel/`)

**`main.tsx`** — the entire app. One large `App` component managing all state, RPC calls, and layout. Key patterns:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adityaongit/idxbeaver](https://github.com/adityaongit/idxbeaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
