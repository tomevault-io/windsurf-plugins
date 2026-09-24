---
trigger: always_on
description: Editor for App Store and Play Store screenshots and preview videos. Next.js 15 **static export** (no server, no API routes) plus a Tauri v2 desktop shell running the same bundle. Projects live in IndexedDB. Desktop adds the MCP server, the embedded-webview AI mode, local AI providers, and loopback OAuth.
---

# Agent guide: Open Screenshot Generator

Editor for App Store and Play Store screenshots and preview videos. Next.js 15 **static export** (no server, no API routes) plus a Tauri v2 desktop shell running the same bundle. Projects live in IndexedDB. Desktop adds the MCP server, the embedded-webview AI mode, local AI providers, and loopback OAuth.

## Map

| What | Where |
| --- | --- |
| Data model, read this first | [src/types/artboard.ts](../src/types/artboard.ts) |
| All state, export, MCP api | [OpenScreenshotGeneratorLayout.tsx](../src/components/open-screenshot-generator/OpenScreenshotGeneratorLayout.tsx) |
| Canvas and element renderers | [CanvasArea.tsx](../src/components/open-screenshot-generator/CanvasArea.tsx), [Artboard.tsx](../src/components/open-screenshot-generator/Artboard.tsx), [elements/](../src/components/open-screenshot-generator/elements/) |
| Templates (101 JSON files) | [public/data/projects/](../public/data/projects/), registered in [templateCategories.ts](../src/lib/templateCategories.ts) |
| Screenshots in, finished designs out | [src/lib/intake/](../src/lib/intake/) + [start/quickstart/](../src/components/open-screenshot-generator/start/quickstart/) |
| AI agent | [src/lib/ai/](../src/lib/ai/) |
| MCP server | [src/lib/mcp/](../src/lib/mcp/) + [src-tauri/src/mcp_server.rs](../src-tauri/src/mcp_server.rs) |
| Translate and fonts | [translation.ts](../src/services/translation.ts), [fontService.ts](../src/services/fontService.ts), [customFonts.ts](../src/services/customFonts.ts), [fontLanguageMatcher.ts](../src/lib/fontLanguageMatcher.ts) |
| Devices, palette, 3D | [deviceRegistry.ts](../src/lib/deviceRegistry.ts), [elementLibrary.ts](../src/lib/elementLibrary.ts), [device3dPresets.ts](../src/lib/device3dPresets.ts) |
| App Preview scenes (whole boards, palette tab 4) | [previewScenes.ts](../src/lib/previewScenes.ts) |
| Video export | [src/lib/video/](../src/lib/video/) |
| Store upload (desktop only) | [src/lib/publish/](../src/lib/publish/) + [publish/PublishDialog.tsx](../src/components/open-screenshot-generator/publish/PublishDialog.tsx) |
| Panels out of the window, and displays | [src/lib/panels/](../src/lib/panels/) + [panels/](../src/components/open-screenshot-generator/panels/) + [panels.rs](../src-tauri/src/panels.rs) |
| Where a project can be saved | [src/lib/account/](../src/lib/account) (the user's own Drive/gists; keeps itself up to date once saved, off by default: [autoSync.ts](../src/lib/account/autoSync.ts)), [src/lib/cloud/](../src/lib/cloud) (ours, the only one that yields a share link, and on by default: [autoSave.ts](../src/lib/cloud/autoSave.ts)) |
| Editing together, live | [src/lib/collab/](../src/lib/collab) (Yjs over WebRTC), signalling in [mcp-relay/src/collab.js](../infra/vps/mcp-relay/src/collab.js) |
| Versions of a project | [src/lib/versions/store.ts](../src/lib/versions/store.ts) + the Dexie `projectVersions` table |
| Dexie, 8 tables | [src/database.ts](../src/database.ts): `projects`, `media`, `operations`, `fonts`, `discoverPosts`, `cloudLinks`, `accountLinks`, `projectVersions` |

## Rules

**Mutating**

1. `handleArtboardsUpdate(next)` is the only door. It repositions boards, writes Dexie, pushes undo. A raw `setArtboards` skips persistence and history. It writes **only** `{ id, name, timestamp, projectData }`, so a new top-level field on `Project` survives until the next keystroke and then vanishes: that is why `ProjectLocalization` is mirrored onto every artboard, and why the cloud-save link lives in its own Dexie table ([src/lib/cloud/links.ts](../src/lib/cloud/links.ts)).
2. Elements render in **two** places: [Artboard.tsx](../src/components/open-screenshot-generator/Artboard.tsx) and `StaticArtboard` in [PreviewDialog.tsx](../src/components/open-screenshot-generator/PreviewDialog.tsx). Miss one and the element vanishes there.
3. Text renders at `fontSize / 0.3` px and ignores `element.scale`. Resize text via `fontSize`. The box clips, so every place a **user** edits text content, family, size, weight or line height folds `fitTextBox` ([textFit.ts](../src/lib/textFit.ts)) into the same update. Template data is never re-fitted.
4. `ArtboardState.position` is derived and overwritten every update. Authoring it does nothing. So is `backgroundImageSlice`. Both are re-derived by `calculateArtboardPositions`, which also puts a board that has no background picture onto the shared one when `backgroundImageApply` is `all` or `span`. That is what makes a shared background stick: an artboard added later, a dropped preview scene and an agent-made board all join it, and a split re-slices itself after any add, delete, duplicate, reorder or resize, without one of those writers knowing the feature exists.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnetdreamer/open-screenshot-generator](https://github.com/dotnetdreamer/open-screenshot-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
