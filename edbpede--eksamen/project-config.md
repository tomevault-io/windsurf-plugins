---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A zero-build static site (`docs/` is the published web root, see `docs/CNAME` → `eksamen.edbpede.net`) that indexes Danish FP9/FP10 examination material. There is **no** `package.json`, lockfile, bundler, or Node toolchain — this is deliberate and documented in the header comments of both CI workflows. Do not introduce one; first-party code is hand-written ES modules and plain CSS loaded directly by the browser.

## Essential Commands

Run everything from the repository root.

```bash
# Serve the site locally. docs/ MUST be the web root — see "Critical Gotchas".
python3 -m http.server 4321 --bind 127.0.0.1 --directory docs

# Full quality gate, identical to the Code Quality workflow.
# SKIP is required on main: no-commit-to-branch is a local guard, meaningless here.
SKIP=no-commit-to-branch prek run --all-files --hook-stage manual

# Single check (fast, targeted) — e.g. just the secret scan or just JSON validity.
prek run gitleaks --all-files
prek run check-json --all-files

# Install the git hooks from prek.toml locally.
prek install
```

There is no unit-test suite. The only automated behavioural check is `.github/workflows/smoke.yml`, which serves `docs/` and asserts that `/index.html` and `/optagelsesprover.html` return real HTML. Reproduce a single page probe locally against the server above:

```bash
curl -fsS -m 10 http://127.0.0.1:4321/optagelsesprover.html | grep -i '<html'
```

## Architecture Overview

Three first-party layers, all client-side:

1. **Landing pages** — `docs/index.html` (FP9/FP10 archive) and `docs/optagelsesprover.html` (gymnasium entrance exams). Both ship an empty `<main class="subject-grid">` and let a module fill it.
2. **List modules** — `docs/js/landing.js` builds subject cards from `docs/js/examList.js` (`subjects` array) plus `docs/js/examScanner.js` (fetches `docs/proever/exam-index.json`). `docs/js/optagelsesprover.js` is a parallel, standalone implementation for the encrypted list; it shares no code with `landing.js`.
3. **Interstitial gate** — every exam link goes to `docs/proever/shared/exam-start.html?exam=<path>`, a "wait for your teacher" confirmation page. `exam-start.js` then redirects to `"../" + examPath`, so **all `path` values in the indexes are relative to `docs/proever/`**.

Styling is a single token set in `docs/css/shared.css` (`:root` custom properties: `--accent`, `--ink`, `--sp-*`, `--font-*`, `--max-w`) plus self-hosted IBM Plex woff2 in `docs/fonts/`. `docs/css/landing.css` pulls it in via `@import url("shared.css")`; exam pages link `shared.css` and `proever/shared/exam-page.css` separately. Add new tokens to `shared.css`, never re-declare colours or spacing inline.

`docs/optagelsesprover.js` gates the entrance-exam list behind a code: `docs/proever/optagelsesprover.enc.json` is an AES-GCM blob with a PBKDF2-SHA256 key (250 000 iterations) derived from the code; a wrong code fails the GCM auth tag. Neither the code nor the exam paths appear in cleartext in the served site. The correct code is remembered per tab in `sessionStorage` under `optagelse_pw`.

## Project Boundaries

| Path | Ownership | Rule |
| --- | --- | --- |
| `docs/index.html`, `docs/optagelsesprover.html`, `docs/css/`, `docs/js/`, `docs/fonts/`, `docs/img/` | First-party | Edit freely. |
| `docs/proever/shared/` | First-party | The shared gate + exam-page stylesheet. Edit here rather than duplicating per exam. |
| `docs/proever/**/index.html` containing `class="doc-page"` | First-party wrapper | Hand-written link page for PDF-only exams. |
| Everything else under `docs/proever/` | Vendored, Ministry-owned | Copy verbatim. Do not reformat, rename, or "clean up". |

`docs/proever/LICENSE` covers the vendored tree; the root `LICENSE` (AGPL-3.0) covers the site code. `prek.toml` excludes `^docs/proever/` from the whitespace, EOF, and line-ending fixers for exactly this reason — see "Licensing" below.

## Common Change Workflows

### Add a PDF-only exam (Dansk læsning, matematik uden hjælpemidler, …)

1. Create `docs/proever/<SUBJECT_DIR>/YYYY-MM-DD_<Descriptor>/` (e.g. `FP9_dansk/2024-12-02_Laesning_Retskrivning`). Danish letters are transliterated: `ae`, `oe`, `aa`.
2. Drop the PDFs in, renamed to lowercase-hyphenated names (`laesning-tekster.pdf`, `matematik-uden-hjaelpemidler.pdf`).
3. Add an `index.html` wrapper. Copy the closest existing one — `docs/proever/FP9_dansk/2024-12-02_Laesning_Retskrivning/index.html` is the canonical shape: root-absolute links for shared assets, relative links for the exam's own files:
   ```html
   <link rel="stylesheet" href="/css/shared.css" />
   <link rel="stylesheet" href="/proever/shared/exam-page.css" />
   ...
   <a class="doc-list__link" href="laesning-tekster.pdf" target="_blank" rel="noopener">
   ```
4. Register it in `docs/proever/exam-index.json` under the subject key, with `path` relative to `docs/proever/`.
5. Bump the hardcoded `Opdateret <date>` in `docs/index.html`.

### Add an interactive (Ministry HTML) exam


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edbpede/eksamen](https://github.com/edbpede/eksamen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
