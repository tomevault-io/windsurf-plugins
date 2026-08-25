---
trigger: always_on
description: Serverless, fully client-side rich-text editor that saves `.odt` (and `.docx`). No backend; all
---

# CLAUDE.md

Serverless, fully client-side rich-text editor that saves `.odt` (and `.docx`). No backend; all
state lives in the browser (localStorage).

**Stack:** Svelte 5 (runes) + TypeScript + Vite. Editor engine TipTap 3 (ProseMirror). ODF export
via `odf-kit` + `fflate` for re-zipping during post-processing.

## Commands

```bash
npm run dev      # dev server (Vite, hot-reload, --host)
npm run build    # production build → dist/
npm run preview  # serve the dist/ build locally
npm run check    # svelte-check type-check (svelte + ts)
npm test         # Vitest suite once (tests/**/*.test.ts)
npm run test:watch   # Vitest in watch mode
npm run test:lo      # LibreOffice round-trip leg only (needs `soffice` on PATH)
npm run test:smoke   # boots the dist/ build in headless Chromium (tests/smoke/run.mjs)
npm run test:coverage  # vitest + v8 coverage over src/ → coverage/index.html
npm run test:parity  # render parity vs LibreOffice (tests/render-parity/README.md)
node scripts/make-thesaurus.mjs   # re-vendor public/thesaurus/ from LibreOffice's MyThes data
node scripts/collect-licenses.mjs # regenerate public/licenses.txt after a dependency change
```

Tests live in `tests/` (outside `src/`, so `svelte-check` ignores them), jsdom via Vitest.
`roundtrip.test.ts` covers the ODF export↔import round trip + a foreign-doc/style-resolver leg;
`lo-roundtrip.test.ts` re-saves through LibreOffice and **self-skips** without `soffice`, so
`npm test`/CI stay green; `corpus.test.ts` round-trips the committed `tests/corpus/`
documents (authored by `render-parity/make-fixtures.mjs`, never by our own exporter);
`fuzz-roundtrip.test.ts` round-trips seeded random documents (`fuzzDoc.ts` generator);
`tests/unit/` holds fast helper tests. All test tooling stays a
`devDependency`. No linter/formatter. CI (`.github/workflows/ci.yml`) runs `check` + `test`.

## Rules

**Comments** — keep them precise and short:

- **Never longer than three lines.** No exceptions — file-header comments, tests and config included. A comment that needs a fourth line is explaining what the code already says; cut it back to what the code can't say.
- **Never describe how the current code differs from an older version** (no "previously…", "this used to…", "changed from…"). Comment only what the current code does and why — git history covers the rest.
- **Don't use Word as a placeholder for "a word processor".** Where LibreOffice does the same thing, describe the behaviour itself ("the caret moves", "the zone auto-grows") instead of "Word-style" / "like Word" / "as in Word". Name a product only where the statement really is about that product: its file format (`w:tblLook`, DOCX), or a quirk only it has — then name both if both apply.

**Commit messages** — a clear description, never an essay: a subject line plus **at most ~8
lines**, however large the change. Probed behaviour, measurements and rationale lists belong
in `docs/architecture/` or the nearest `CLAUDE.md`, not repeated in the history.

**IMPORTANT: no real-world document's name anywhere in the repo, and a commit message
names no document at all** — not even "the thesis": describe the fix and its measurement,
never the file it helped. Detail in `tests/render-parity/README.md`.

**IMPORTANT: never introduce a default only this editor has.** Both importers suppress values equal
to the defaults, so an editor-only default is indistinguishable from a failed style resolution and
silently lands in every imported document as direct formatting. The defaults follow **LibreOffice**
(we save `.odt`, so LO wins the ties against Word's Calibri 11pt / 8pt after / 2.54cm):

- Paragraph spacing **0** — no `margin-top`/`-bottom` on `p`, `ul`/`ol` or the table wrapper. Blank lines come from the document's own empty paragraphs.
- Body **Liberation Serif 12pt**, single line spacing; page margins **2cm**; tab/indent step **1.25cm**.
- Headings sans (`HEADING_FONT` = Arial, bundled `@font-face` maps it to metric-identical Liberation Sans), sizes/margins in `HEADING_STYLE_OVERRIDES` (`export/odt.ts` — see `src/lib/export/CLAUDE.md`).
- Liberation Serif TTFs are bundled (`src/assets/fonts/`) and metric-identical to Times New Roman, so editor, LibreOffice and Word share the same metrics. `utils/fontDetect.ts` filters `CANDIDATE_FONTS` to what's installed for the font picker.

**IMPORTANT: keep the layout constants in sync** between `editor/extensions/pageBreaks.ts`,
`components/Editor.svelte` and `styles/editor.css` (`PAGE_HEIGHT` 1123px, `PAGE_GAP` 20px, the
`--user-page-*`/`--user-margin-*` custom properties) — details in `docs/architecture/pagination.md`.

**Headless browser testing** — this container is linux **arm64**. Never `puppeteer` /
`@puppeteer/browsers`: they fetch an x86-64 Chrome that cannot run here. Use `playwright-core` +
its Chromium; full recipe in `docs/headless-testing.md`. Driving the live app is the only way to
verify rendering, layout or NodeView behaviour — there is no DOM test suite for it.

**Naming** — components `PascalCase.svelte`, every `.ts` module `camelCase`; extension files are
named by feature (`image.ts`, `indent.ts`), not `XyzExtension.ts`.

**Documenting a change** — a feature that changes how this codebase behaves gets documented where

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stffnb/edentext](https://github.com/stffnb/edentext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
