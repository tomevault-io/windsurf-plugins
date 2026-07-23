---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Obsidian plugin (`attachment-management`) that customizes attachment paths per-note using variables (`${notepath}`, `${notename}`, `${parent}`, `${originalname}`, `${date}`, `${md5}`) and auto-renames on file events. Targets Obsidian's plugin API; entry point is [src/main.ts](src/main.ts), bundled into a single `main.js`.

## Commands

Package manager is **pnpm** (declared in [package.json](package.json)). The working directory *is* a live Obsidian plugin folder under `Obsidian Sandbox/.obsidian/plugins/obsidian-attachment-management`, so `pnpm run build`/`dev` produces `main.js` in place and the sandbox vault loads it directly.

- `pnpm run dev` — esbuild watch mode (inline sourcemap).
- `pnpm run build` — `tsc -noEmit` typecheck then production esbuild bundle (`BUILD_ENV=production`, no sourcemap).
- `pnpm run lint` — ESLint with `--fix` over `src/`.
- `pnpm run format` — Prettier write over `src/**/*.ts`.
- `pnpm run version` — bumps `manifest.json` + `versions.json` via [version-bump.mjs](version-bump.mjs); invoked by `standard-version`.
- No test suite is configured.

## Architecture

### Event flow (the non-obvious part)

Obsidian's API does not fire `paste`/`drop` for canvas, and on markdown the file-creation order matters. The plugin handles attachment creation in **two stages** registered in [src/main.ts](src/main.ts):

1. `vault.on("create")` — pushes the new TFile into `plugin.createdQueue`, after filtering by mtime/ctime gap (>1s ⇒ assumed copy from OS, ignored), extension (markdown/canvas ignored), and `excludeExtensionPattern`.
2. `vault.on("modify")` — when the note's body is modified (Obsidian writing the link), reads the note via `cachedRead` (NOT `adapter.process`, which would re-write the file and cause the editor cursor to jump — see commit ff49d09), checks whether the markdown link or canvas reference for the queued attachment is present, and only then calls `CreateHandler.processAttach` to rename/move it.

`rename` and `delete` events also drive auto-rename and empty-folder cleanup; `rename` updates per-file/per-folder override settings when their paths change.

### Core modules

- [src/main.ts](src/main.ts) — `AttachmentManagementPlugin`: event wiring, command registration (`Rearrange linked attachments`, `Rearrange all linked attachments`, `Overriding setting`, `Reset Override Setting`, `Clear unused original name storage`).
- [src/create.ts](src/create.ts) — `CreateHandler.processAttach`: post-creation rename/move using the resolved setting.
- [src/arrange.ts](src/arrange.ts) — `ArrangeHandler` + `RearrangeType` (`ACTIVE` | `LINKS` | `FILE`): scans linked attachments across the vault/active note/single file and rearranges them. Marked experimental in the README.
- [src/override.ts](src/override.ts) — resolution of per-file → nearest-parent-folder → global setting; `settings.overridePath` is keyed by path and re-keyed on rename.
- [src/exclude.ts](src/exclude.ts) — path-based exclusion (`excludedPaths`, `excludeSubpaths`).
- [src/utils.ts](src/utils.ts) — file-type predicates (`isMarkdownFile`, `isCanvasFile`, `isImage`, `isPastedImage`, `isAttachment`), regex extension match, `md5sum`.
- [src/settings/metadata.ts](src/settings/metadata.ts) — `Metadata` class: turns a note path into `{path, name, basename, extension, parentPath, parentName}` and resolves `getAttachmentPath` / `getAttachFileName` by substituting the `${...}` variables. **Extension overrides are applied here first**, then falls back to per-file/folder/global setting. `${date}` is anchored to the attachment's `stat.ctime` (not `moment()`) so re-arranges and re-renames stay idempotent — `getAttachFileName` takes the source `TFile` for this reason.
- [src/settings/settings.ts](src/settings/settings.ts) — `AttachmentManagementPluginSettings`, `DEFAULT_SETTINGS`, `SETTINGS_TYPES` enum (`GLOBAL` / `FOLDER` / `FILE`), and the settings tab UI.
- [src/lib/originalStorage.ts](src/lib/originalStorage.ts) — persists `${originalname}` mappings as `{n, md5}` records in `data.json` so the original filename survives later renames. Cleanup is manual via the `Clear unused original name storage` command (matches by md5 of current vault attachments).
- [src/lib/path.ts](src/lib/path.ts) — local minimal path helpers (do not import Node `path`; plugin must work on mobile).
- [src/lib/linkDetector.ts](src/lib/linkDetector.ts) — scans markdown/canvas content for attachment links during rearrange.
- [src/lib/deduplicate.ts](src/lib/deduplicate.ts) — `deduplicateNewName` for collision handling on create/rename.
- [src/lib/log.ts](src/lib/log.ts) — `debugLog` gated on `BUILD_ENV !== "production"` (defined by [esbuild.config.mjs](esbuild.config.mjs)).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trganda/obsidian-attachment-management](https://github.com/trganda/obsidian-attachment-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
