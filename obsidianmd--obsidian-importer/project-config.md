---
trigger: always_on
description: Imports notes from other apps into an Obsidian vault.
---

# Obsidian Importer

Imports notes from other apps into an Obsidian vault.

## Project structure

- `src/main.ts` — Plugin entry point and desktop import modal
- `src/importers.ts` — Importer registry, groups, names, icons, and help links
- `src/importer-flow.ts` — Shared import screens and the `ImporterShell` contract
- `src/importer-setting-tab.ts` — Settings host for the shared flow
- `src/progress-ui.ts` — `ImportProgressUI`, the progress screen driven by an `ImportContext`
- `src/format-importer.ts` — Base importer: steps, file pickers, templates, output folders, duplicate handling, note writes, and attachments
- `src/note-template.ts`, `src/note-template-configurator.ts` — Knap template rendering and configuration
- `src/list-properties.ts` — Final normalization of Obsidian's built-in list properties
- `src/formats/<name>.ts` — Vault-facing importer code
- `src/formats/<name>/` — Conversion code split out so it can run without a vault, when the format has such a split
- `src/filesystem.ts` — Shared filesystem and Node-module seam injected by tests
- `src/encoding.ts` — Encoding detection used by every `PickedFile.readText()` implementation
- `src/util.ts` — Shared parsing, sanitizing, frontmatter, and unique-path helpers
- `src/outline.ts`, `src/block-refs.ts`, `src/markdown.ts` — Reusable outliner primitives described below
- `tests/shims/` — Obsidian API, DOM, runtime-prototype, vault, picker, and zip substitutes used outside the app
- `tests/<importer>/` — Fixtures, tests, and recorded output under `expected/`

## Build and test

- `npm run build` — Typecheck and create the production `main.js` bundle
- `npm test` — Run every test
- `npm test -- notion` — Run one importer suite; several suite names may be supplied
- `npm run typecheck` and `npm run lint:check` — Both must pass
- `npm run lint:review` — Run the stricter configuration used for Obsidian community-plugin review

**`eslint-disable` does not suppress findings in the community review.** Resolve the finding instead.

## Import flow hosts

`ImporterFlow` owns the import screens and is independent of its surrounding window. An `ImporterShell` supplies its content and container elements, navigation chrome, focus behavior, button-bar placement, finish behavior, and a way to bring the flow forward.

`ImporterModal` hosts the ribbon and command flow on desktop. On mobile, both entry points open `ImporterSettingTab`. Settings renders each screen beyond the format list as a `SettingPage`. On phones, Settings owns the title-bar back button, so `ownsBackButton` prevents the flow from drawing a second one; larger screens and the modal put Back beside Help.

Screen depth is the number of screens beyond the format list: the list is 0, a grouped method picker is 1, and importer steps continue from there. The Settings shell opens or closes pages until its stack matches that depth. A running import has no screen behind it, so Back leaves the flow and unwinds the page stack.

Each screen passes its action row to `adoptButtonBar`; the format list passes `null`. In Settings, the shell keeps the bar outside the scrolling, animated `SettingPage` and refills it in place so it neither scrolls away nor jumps during a page transform. The shell also toggles `has-button-bar` so the page's scrollbar ends above the bar. Do not replace that class with `:has()`; community-plugin CSS may not use `:has()`.

An importer with a separate configuration screen overrides `configures`, which changes the last source step from Import to Continue. The run stays on the configuration page and redraws its `ImportProgressUI` there. Construct that UI in the configuration screen's container; otherwise its initial render flashes beneath the configuration before the run redraws it.

File-export importers begin with `addExportSetting`. If finding the export needs extra help, wrap that row with `addInstructions`. Connected importers begin with the credential, account, or folder they actually require rather than a redundant service-name row. Help appears on every screen and comes from the registry's single `helpPermalink`, exposed through `ImporterHost`.

Settings pages require Obsidian 1.13, which is why `manifest.json` declares `minAppVersion: 1.13.0`. The published `obsidian` package is still 1.12.3, so the missing 1.13 `SettingPage` APIs are declared in `src/augment.d.ts`.

A host calls `flow.detach()` when its UI disappears and `flow.attach()` when it returns. Detaching leaves an active import running and reports progress through a notice; attaching redraws its saved screen. `flow.leave()` returns to the format list while preserving the background run in `awayFrom`. `flow.dispose()` permanently tears down the flow and cancels its work; closing the modal uses it.

`ImporterSettingTab.pageClosed()` defers its answer by one microtask so Settings can finish changing the page stack. It treats a single remaining stack change as Back and ignores wholesale page or tab teardown, preserving the flow for reopening.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obsidianmd/obsidian-importer](https://github.com/obsidianmd/obsidian-importer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
