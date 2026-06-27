---
trigger: always_on
description: This file is the handoff document for future coding sessions. Read it before changing code. Keep it factual and update it whenever architecture, storage, provider behavior, or release workflow changes.
---

# Zotero-Cat Implementation Notes

This file is the handoff document for future coding sessions. Read it before changing code. Keep it factual and update it whenever architecture, storage, provider behavior, or release workflow changes.

## Project Identity

- Product name: `Zotero-Cat`
- Package name: `zotero-cat`
- Add-on display name: `Zotero-Cat`
- Add-on ID: `zotero-cat@qianjindexiaozu.dev`
- Add-on namespace: `zoterocat`
- Zotero global instance: `Zotero.ZoteroCat`
- Zotero pref prefix: `extensions.zotero.zoterocat`
- Repository path on the current machine: `/Users/qianjindexiaozu/projects/Zotero-Cat`
- Remote: `git@github.com:Zotero-Cat/Zotero-Cat.git`
- Domain owned by the maintainer: `zoterocat.org`
- License: `AGPL-3.0-or-later`

Zotero-Cat is independent from Zotero. Public docs should include a non-affiliation statement.

## Current Position

Zotero-Cat is a Zotero item-pane assistant. It uses Zotero's official `ItemPaneManager.registerSection` API, so it appears as a section in Zotero's existing right item pane. It does not replace Zotero's native right sidebar and does not try to own the full pane.

The current implementation covers MVP, Zotero context injection, streaming chat UX, per-item history, persistence, internal diagnostics, Phase 3.5 engineering quality, repository-side Phase 4 release preparation, optional web search tooling, tool-action orchestration, session export/rename/favorite controls, and experimental PDF tool agency. PDF tool agency currently includes `read_pdf`, `list_annotations`, annotation proposal generation, Accept / Reject / Accept All / Reject All review cards, optional auto-apply, and Zotero annotation create/update/delete wrappers. Structure work moved model metadata parsing, conversation persistence, item scoping, retry classification, shared message types, web search logic, tool-action parsing, PDF text extraction, annotation persistence, and proposal state out of the item-pane UI file. Release docs, changelog, provider setup notes, privacy notes, and the direct GitHub release workflow are present. Public Markdown intended for users has English and Chinese versions; `README.md` remains the English GitHub homepage and links to `README.zh-CN.md`.

The current public release is `v0.3.1`. It patches PDF quote matching for GLM-style trailing ellipses and model-emitted line-break hyphen spacing while keeping the `v0.3.0` tool-call hardening work intact. The public release asset is `zotero-cat-v0.3.1.xpi` under the version tag. The special GitHub release tag named `release` is used only for updater manifests and should remain marked as pre-release and not Latest. Zotero 10 beta compatibility is still not declared; keep `strict_max_version` at `9.*` until the current Zotero beta line passes the manual checklist.

## Development Environment

Use Node.js 24 LTS.

Version files:

- `.nvmrc`: `24`
- `.node-version`: `24`
- `package.json` engines: `>=24 <25`

Core commands:

```bash
nvm use
npm install
npm run lint:check
npm run build
npm test
npm start
```

`npm test` runs `zotero-plugin test --exit-on-finish`. This matters because the scaffold test runner otherwise keeps Zotero test processes alive after the suite finishes.

CI uses `actions/setup-node@v4` with `node-version-file: .nvmrc`, then `npm ci`.

## Main Architecture

### Add-on bootstrap

- `src/index.ts` creates `Zotero.ZoteroCat` if it does not exist.
- `src/addon.ts` holds shared add-on state and hook references.
- `src/hooks.ts` handles startup, shutdown, main-window loading, preference pane registration, and reader selected-text event registration.
- `zotero-plugin.config.ts` reads package config and passes name, ID, namespace, prefs prefix, and script output path to `zotero-plugin-scaffold`.

### Agent UI

Primary file: `src/modules/agent/section.ts`.

Responsibilities:

- Register and unregister the item-pane section.
- Render the full chat UI.
- Manage runtime UI state.
- Handle send, stop, retry, streaming output, copy feedback, internal diagnostics, model selection, tool toggles, and session controls.
- Coordinate conversation loading and saving through `conversationFileStore.ts`, and keep in-memory session selection/mutation in `conversationRuntime.ts`.
- Render annotation proposal batches and route accepted proposals through the PDF annotation tool wrappers.

Important UI decisions:

- The chat panel uses a fixed height derived from 85 percent of the visible page.
- The input composer belongs at the bottom of the panel.
- The session selector stays at the top of the chat area.
- History uses a native dropdown, not a custom lazy list.
- The dropdown shows up to 8 recent conversations for the current Zotero item.
- Custom context, context preview, and diagnostics disclosure panels are not rendered in the chat controls.
- The item-pane UI does not inject custom context text.
- Session controls support export, rename, and favorite.
- PDF tools are off by default and exposed from the chat controls.
- PDF write actions must become proposal cards first; direct model-driven annotation mutation is not allowed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zotero-Cat/Zotero-Cat](https://github.com/Zotero-Cat/Zotero-Cat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
