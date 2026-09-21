---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Obsidian plugin that reviews CriticMarkup suggestions (typically authored by an AI) in a side panel — accept/reject/reply. No sidecar state; everything lives as `{++…++}`, `{--…--}`, `{~~old~>new~~}`, `{>>comment<<}`, `{==highlight==}`, `{=+ai-added+=}` directly in the markdown.

## Commands

```sh
npm install              # one-time
npm run dev              # esbuild watch -> main.js (with inline sourcemaps)
npm run build            # tsc --noEmit + esbuild production bundle
npm run typecheck        # tsc --noEmit -skipLibCheck
npm test                 # runs all six test files sequentially
node test/parser.test.mjs        # run a single test file
```

Tests are plain Node ESM scripts (`.mjs`) under `test/` — no test framework. They import compiled TS via Node's TS loader path or by re-implementing fixtures; check an existing test before adding one.

To load the dev build into Obsidian: symlink or copy `main.js`, `manifest.json`, `styles.css` into `<vault>/.obsidian/plugins/track-changes/`.

## Architecture

Entry point `src/main.ts` is the `Plugin` subclass. It wires four things into Obsidian and owns nothing else of substance:

1. **Right-side panel view** (`src/panel/view.ts`, `REVIEW_VIEW_TYPE`) — the review UI. `main.ts` constructs a `PanelHost` adapter so the panel never imports the `Plugin` directly; the panel calls back through `host.applyEdits`, `host.revealOffset`.
2. **CodeMirror 6 decoration extension** (`src/editor/decorations.ts`) — inline highlighting of CriticMarkup ranges in Live Preview / Source mode. Click handler routes back into `main.handleInlineClick` which opens the panel and focuses the offset.
3. **Reading-mode post-processor** (`src/reading.ts`) — renders markup in preview mode either as accepted preview or side-by-side, based on settings.
4. **Commands**: open panel, finalize for publish (`src/finalize.ts`), five manual-authoring commands (`src/authoring.ts`).

### Data flow: parse → edits → rebase → apply

- `src/parser.ts` scans source text and emits a `ParseResult` with `nodes` (the six CriticMarkup kinds) and `threads` (adjacent `{>>…<<}` blocks group). **All six kinds** carry an optional metadata prefix — a run of space/tab-separated `key="value"` pairs (HTML-attribute flavored), no leading whitespace, placed between the outer `{` and the sigil. A key is `[A-Za-z][\w-]*` (lowercased on lookup); a value is double-quoted and **may not contain `"`, `{`, `}`, or a newline** (everything else — spaces, `;`, `=`, `:`, `+`, `~`, `<`, `>`, `-`, `.`, `,`, `'` — is allowed). Surfaced on `BaseNode` as `metaAttrs: Record<string, string>` (every key, lowercased, empty values dropped, first occurrence wins — carries `author`/`date` **and any future key** like `status` with no parser change) plus the typed accessors `metaAuthor: string | null`, `metaDate: string | null` (display-only, never validated/sorted), `metaRaw` (the exact prefix consumed incl. any trailing space, `""` if none), and `innerFrom`/`innerTo` (payload bounds after the prefix+sigil — render paths use these, not `from+3`/`to-3`). `from`/`to`/`raw` span the whole outer-brace-to-outer-brace including the prefix, so accept/reject/finalize strip it for free. Comments also expose `authorName: string | null` — the legacy captured `<Name>:` body prefix (original casing) or `null`. Parsing is always-on (no toggle). **The value class forbidding `"`/`{`/`}`/newline is the corruption defense** (replacing the old mandatory trailing `;`): a truncated/streamed value (`date="2026`) finds no closing `"` before the next brace or line break, so the pair fails, the prefix collapses to `""`, and the mark fails to form *locally* instead of swallowing downstream text. There is **no nesting guard**: a mark whose body contains a nested mark (`{--a {>>b<<} c--}`, prefixed or not) collapses into the *outer* mark via the overlap-drop pass — the inner is part of the added/deleted text. The quoted value class (no `"`/`{`/`}`/newline inside a value) makes a *prefixed* straddle structurally impossible, so no guard is needed; a legit single brace in prose stays inside its one mark. (The reading-view post-processor strips the prefix from rendered preview within a single DOM text node — a value containing markdown can split it across nodes and leak; plain `author`/`date` values are unaffected.) **Code blocks are skipped** — markup inside fenced (```` ``` ````, `~~~`), indented (4-space / tab), or inline-backtick code is left alone. The sixth kind `aitext` (`{=+…+=}`, sigil "highlight + addition") marks AI-*inserted* text and is **visual-only**: no panel card, not clickable (decorations omit `data-tc-offset`), rendered as a rainbow `.tc-aitext` span in Live Preview and reading view (reading mode strips the sigils but *wraps* the body — the one render path that adds an element rather than only deleting tokens). `finalizeEdits` strips it (keeping the text) when `finalize.stripAiText` is true (default).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [philphilphil/obsidian-track-changes](https://github.com/philphilphil/obsidian-track-changes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
