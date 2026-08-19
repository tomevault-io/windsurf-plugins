---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Redactix is a zero-dependency, vanilla-JavaScript WYSIWYG editor distributed as native ES Modules. There is **no build system, no package manager, no test suite, and no transpilation** — files are served directly to the browser. The repo is set up as an OpenServer document root; opening [index.html](index.html) in a browser via the local web server (or any static server pointed at this directory) is how you "run" the project. Image and (opt-in) native video upload/browse/delete relies on two PHP scripts at the repo root.

## Architecture

The editor wraps a `<textarea>` with a contenteditable `<div>` and keeps the textarea synced so any backend that already reads `<textarea>.value` works unchanged.

### Boot flow ([redactix/Redactix.js](redactix/Redactix.js))

1. `new Redactix({ selector, ... })` finds matching textareas.
2. For each textarea it constructs a `RedactixInstance`, which:
   - Builds a `.redactix-wrapper` containing `Toolbar` + `.redactix-editor` (contenteditable).
   - Hides the original textarea and exposes the instance back on the DOM node as `textarea.redactix` (this is the public API surface — `getContent()`, `setContent()`, `sync()`, `setTheme()`, `destroy()`).
   - Instantiates `Editor` (core editing/paste/structure logic), `Selection`, `Modal`, then iterates `modulesConfig` and calls `new ModuleClass(this).init()` on each.
   - Asks the toolbar to gather buttons from each module via `module.getButtons()`.

The `modulesConfig` array in the `Redactix` constructor ([Redactix.js](redactix/Redactix.js)) is the registration point for modules. Order matters for some modules (e.g., `History` is first so it can wrap subsequent module actions).

### Teardown (listener registry + destroy)

Modules must register `document`/`window` listeners via `instance.listen(target, type, handler, options)` and other persistent resources (MutationObservers, `<body>` portals like the BlockControl menu) via `instance.onDestroy(cb)`. `RedactixInstance.destroy()` cancels the pending sync frame, removes every registered listener, runs the cleanups, removes the wrapper, and un-hides the original textarea (which can then be re-initialized). Listeners on elements inside the wrapper don't need registration — they die with `wrapper.remove()`. **If you add a module with a global listener, route it through `instance.listen`.**

### Module system ([redactix/core/Module.js](redactix/core/Module.js))

Every feature is a class extending `Module`. A module receives the `RedactixInstance` and can:
- Access the contenteditable via `this.instance.editorEl` and the core editor via `this.editor` / `this.instance.core`.
- Read config via `this.instance.config` (this includes `liteMode`, `uploadUrl`, `browseUrl`, `videoUploadUrl`, `videoBrowseUrl`, `allowVideoDelete`, `calloutPresets`, `quotePresets`, `predefinedClasses`, `maxHeight`, `theme`, `gapInsertHandle`, `i18n`).
- Return toolbar buttons from `getButtons()` (`{ name, icon, title, action, active? }`).
- Translate strings via `this.t('namespace.key')`.
- Override `hideUI()` if the module renders floating overlays positioned over the editor (block handles, gap "+", menus, panels). HtmlMode calls it on every module when the visual editor is hidden — otherwise stale absolutely-positioned overlays linger on top of the code editor.

To add a new feature: create a file in [redactix/modules/](redactix/modules/), extend `Module`, then add the class to the `modulesConfig` array in [Redactix.js](redactix/Redactix.js).

URL / rel / target / inline-HTML sanitisation is centralised in [redactix/core/dom-utils.js](redactix/core/dom-utils.js) (`sanitizeUrl`, `sanitizeImageSrc`, `composeLinkRel`, `sanitizeInlineHtml`, `normalizeInlineSynonyms` — the latter canonicalises `<strong>`→`<b>`, `<em>`→`<i>`, `<strike>`→`<s>` on paste, content load, HTML-mode return and on the sync clone, so saved output never mixes synonym tags). **Every** path that writes user input into the document goes through it — both the paste sanitizer and all modal forms (Image, Gallery, Video, Embed, QuoteCard author, Link, FloatingToolbar). Captions / author names accept inline HTML, but only through `sanitizeInlineHtml`. When adding a module with a URL or rich-text field, use these helpers instead of writing values raw.

### Lite mode

`liteMode: true` is the comments/forum profile. It is enforced in two places:
- The instance-config assembly in [Redactix.js](redactix/Redactix.js) (`init()`) — strips `uploadUrl`/`browseUrl`/`videoUploadUrl`/`videoBrowseUrl` from the per-instance config so upload paths are inert.
- The wrapper gets the `redactix-lite-mode` class; CSS hides the toolbar and individual modules check `this.instance.config.liteMode` to disable features (advanced link options, attribute editing, base64 paste, counter, etc.).

When changing module behavior, check whether lite mode needs a branch.

### Block gap insert handle


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fedorananin/redactix](https://github.com/fedorananin/redactix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
