---
trigger: always_on
description: Develop **K-Plex (Knowledge Plex)** as a dedicated React application inside Obsidian while preserving the relationship semantics, ontology model and useful settings compatibility of classic ExcaliBrain.
---

# AGENTS.md

## Mission

Develop **K-Plex (Knowledge Plex)** as a dedicated React application inside Obsidian while preserving the relationship semantics, ontology model and useful settings compatibility of classic ExcaliBrain.

K-Plex is not an Excalidraw extension. Excalidraw and Dataview must not be runtime requirements.

The plugin ID is **`k-plex`** so K-Plex can coexist with legacy ExcaliBrain during migration.

## Build contract

Baseline Node.js: **22.22.2**.

```bash
npm i
npm run build
```

Use `npm run dev` for watch-mode development.

A change is not complete if the real repository does not build successfully.

Installable output must be written to `./dist/`:

- `dist/main.js`
- `dist/manifest.json`
- `dist/styles.css`

Do not claim a successful build from a stub-only/type-harness check. When Obsidian APIs are involved, validate against the actual installed `obsidian` type package.

## Obsidian API discipline

This project has already lost time to invented/assumed APIs. Do not guess Obsidian methods.

Rules:

1. Check the installed Obsidian type declarations and current official documentation before using an unfamiliar API.
2. Prefer public typed APIs.
3. Example: `MetadataCache.getFileCache(file)` is valid and `getAllTags(cache)` is exported; do **not** invent `metadataCache.getTags()`.
4. Use Obsidian `Modal` for centered modal dialogs.
5. Use `workspace.getLeaf("window")` for pop-out workflows when supported by the installed API.
6. Use the Obsidian declarative settings API for settings UI.
7. All plugin UI icons must be Lucide icons obtained through Obsidian `getIcon()` (or a thin React wrapper around it). Do not ship hand-coded icon SVGs or unrelated icon libraries.
8. Moment is host-provided by Obsidian. Do not runtime-import `moment` or call the `moment` export from `obsidian`; production code should use Obsidian's `window.moment` through narrow local typing. Tests may install a Moment test double on `window`.

## Non-negotiable compatibility rules

1. Preserve classic ExcaliBrain ontology semantics unless a deliberate migration/change is documented.
2. Explicit document-property relationships take precedence over inferred/body relationships.
3. Preserve parent / child / left-friend / right-friend / previous / next reconciliation behavior.
4. Preserve support for Markdown notes, attachments, folders, tags, URLs and virtual/unresolved nodes.
5. Preserve legacy style inheritance as closely as possible without relying on Excalidraw rendering.
6. Preserve/migrate legacy persisted settings instead of silently reinterpreting them.
7. Migrate old `hierarchy.friends` to `hierarchy.leftFriends`.
8. If legacy ExcaliBrain is installed and running, automatically import compatible settings the first time K-Plex runs in that vault. Keep a manual import path as well.
9. Folder and tag nodes may be central nodes, but drag-link creation/relinking involving folder/tag endpoints is disabled.
10. A Markdown document property overrides an equivalent relationship discovered in body text. This rule is important for deterministic relinking.

## Architecture boundaries

- `src/index/` owns graph construction, relationship semantics, search data, caches and compatibility.
- `src/ui/` owns React presentation and interaction.
- `src/settings.ts` owns settings schema/defaults, persistence compatibility and migration.
- `src/main.ts` owns Obsidian lifecycle, commands, workspace/window/leaf integration and rebuild scheduling.

Do not reimplement relationship classification inside React components. UI code should consume normalized index APIs.

Keep Obsidian-specific side effects behind clear boundaries. Presentational components should not reach deeply into workspace/vault APIs when plugin/index services can perform the operation.

## Performance is a product requirement

The plugin must remain responsive in vaults with 20,000+ files and 100,000+ graph/search entries.

### Startup

Obsidian emits large numbers of file events while initializing a vault. Never trigger a full rebuild for every startup `vault:create` or metadata event.

Required strategy:

- wait until layout is ready and metadata is sufficiently stable before the initial full graph build
- register/coalesce normal rebuild listeners only after startup initialization is under control
- mark the index dirty on changes and skip periodic refreshes when it is clean
- collapse event bursts into at most one rebuild/catch-up rebuild

### Persistent parsing cache

Markdown body parsing is expensive and should not repeat on every startup.

- persist parsed inline-field / external-link body metadata in vault-local storage
- key cache records by path + modification time (or an equivalent safe invalidation key)
- restore the cache before the first graph build
- write cache updates lazily/debounced

### Worker boundary

CPU-heavy Markdown-body text parsing may run in a Web Worker.

Do not attempt to use Obsidian APIs inside the worker. Vault/metadata access, graph mutation and Obsidian object handling remain on the main thread.

### Derived-data caching

Avoid repeated global work while rendering one scene.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zsviczian/kplex](https://github.com/zsviczian/kplex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
