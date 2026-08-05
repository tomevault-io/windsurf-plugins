---
trigger: always_on
description: This is an open-source repo. Do not put private/internal issue IDs in branch names, PR titles, PR descriptions, changelog entries, or public comments.
---

# AGENTS.md

## Scope

This is an open-source repo. Do not put private/internal issue IDs in branch names, PR titles, PR descriptions, changelog entries, or public comments.

## Engineering Judgment

When the user leaves implementation details open, you choose conservatively and in sympathy with the codebase already in front of you:

- Read the surrounding UI framework before changing it: component, manager, factory, SCSS, and spec patterns usually show where a change belongs.
- You prefer the repo’s existing patterns, frameworks, and local helper APIs over inventing a new style of abstraction.
- Use structured APIs, typed helpers, and existing parsers instead of ad hoc string manipulation when the repo or platform already provides them.
- Keep changes local unless shared behavior already exists or the same rule is repeated in more than one place.
- Add abstractions only when they remove real duplication, document a stable public surface, or match an established local pattern.
- Treat exported config, components, `UIFactory` layouts, `UIManager` APIs, and TypeDoc-visible types as public API.
- Let verification match risk: focused specs and targeted checks for local changes; browser and mobile SDK verification when behavior crosses UI or platform boundaries.

## Editing Guardrails

- You may be in a dirty git worktree, often with local demo changes. Preserve existing user changes and never revert them unless explicitly requested.
- If existing changes affect files you need to touch, read them carefully and work with them. Ask only when they make the task impossible to complete.
- Keep manual edits narrow and reviewable. Avoid unrelated cleanup, formatting churn, or opportunistic refactors.
- Add comments only for non-obvious behavior contracts, precedence, lifecycle, browser/platform quirks, or public API expectations.
- Avoid destructive Git commands such as `git reset --hard` or forced checkouts unless the user explicitly asks for them.

## UI Structure

- `UIFactory` is the composition layer. Default layouts, variant-specific layout wiring, and feature-to-component wiring should live there instead of making low-level components know about a specific feature.
- Keep `UIFactory` declarative. It should compose layouts and wire components together, not become the place for player-state business logic or event behavior.
- `UIManager` owns player/UI lifecycle and variant switching. `UIInstanceManager` owns the active UI instance. Keep variant resolution, active UI state, and cleanup concerns in those layers.
- Components should stay reusable. A component may depend on generic framework primitives such as `Component`, `Container`, `Button`, `Panel`, `SettingsPanel`, or `EventDispatcher`, but should not know about unrelated sibling features just to make one layout work.
- `Component` classes own DOM and component-local behavior. Shared state and cross-component coordination belong in manager/state layers, not in sibling components reaching into each other.
- Panel-like UI should use the existing page/item/container patterns. Build root pages explicitly; do not hide page creation or navigation structure in implicit wrapper logic.
- Interactive rows inside panels should have one clear focus target. If a row wraps a nested button/toggle/select control, make the row and nested control interaction explicit so click, keyboard, ARIA state, and spatial navigation do not fight each other.
- Document/root listeners must be tied to the active UI lifecycle and released again. Keep stable handler references, unsubscribe in `release()` or the matching inactive/source-unloaded path, and avoid global singleton UI state. This repo supports multiple UI instances, UI variants, and Shadow DOM.

## Player API And Mobile SDKs

- The player is the source of truth for playback state. UI code should observe the player and recompute from player APIs or events instead of inferring playback state from UI variant, controlbar state, or component internals.
- For runtime player enums, events, and classes, prefer the active `player.exports.*` surface so the UI uses the same player instance it is attached to. Type-only imports from `bitmovin-player` are common in this repo, but do not add runtime player imports that can duplicate player code in the UI bundle.
- Use the public [Player Web API reference](https://cdn.bitmovin.com/player/web/8/docs/index.html) as the first source of truth for `player.*` and `player.exports.*` behavior.
- This UI is also used by the Bitmovin Android and iOS SDKs. Browser automation cannot fully cover those WebView/native-bridge environments, so changes that touch player APIs, platform/browser APIs, input behavior, layout, or generated markup need manual mobile SDK verification.
- The UI is implemented against the Player Web API. Before using a new `player.*` or `player.exports.*` API, verify that the Android and iOS SDK bridges expose it and that return values, events, timing, error behavior, and `undefined`/`null` cases match the web player closely enough for the UI code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitmovin/bitmovin-player-ui](https://github.com/bitmovin/bitmovin-player-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
