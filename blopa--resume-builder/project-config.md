---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Human-facing docs live in [README.MD](README.MD).
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Human-facing docs live in [README.MD](README.MD).

## Project Overview

Resume Builder is a **Gatsby 3 static site** (React 17 + Material UI 4) that turns resume data into printable resumes. It is deployed to GitHub Pages at [resume-builder.js.org](https://resume-builder.js.org/).

**There is no backend.** Spreadsheets, `.json` uploads, and GitHub-hosted resumes are all fetched and parsed in the browser. Never introduce a server-side dependency, an API key, or anything that assumes a running server.

## Setup & Commands

Requires Node **20.x** (see `.nvmrc`; run `nvm use`).

| Command          | Purpose                                                                  |
| ---------------- | ------------------------------------------------------------------------ |
| `npm install`    | Install dependencies.                                                    |
| `npm run start`  | Dev server at `http://localhost:8000` (alias for `develop`).             |
| `npm run build`  | Production build into `public/`.                                         |
| `npm run serve`  | Serve the production build.                                              |
| `npm run clean`  | Clear `.cache/` and `public/` — do this first when Gatsby behaves oddly. |
| `npm run format` | Prettier over the codebase. Run before committing.                       |

**There is no test suite.** `npm test` is a placeholder that prints a message and exits 1 — this is expected, not a broken build. Do not report it as a failure, and do not "fix" it by making it exit 0. If you add real tests, wire up the runner properly.

**There is no standalone lint script.** ESLint runs automatically via `gatsby-plugin-eslint` during `develop`/`build`. To lint directly: `npx eslint src --ext .js,.jsx`.

## Critical Conventions

### Line endings are CRLF

`.gitattributes` forces `eol=crlf` for `*.js`, `*.jsx`, `*.json`, `*.md`, and `.prettierrc` sets `"endOfLine": "crlf"`. Writing LF files will produce whole-file diffs that bury the actual change. If you touch a file and the diff shows every line changed, this is why — run `npm run format` and re-check.

### Code style

Enforced by Prettier (`.prettierrc`) — 4-space indent, single quotes, semicolons, 120-char width, `trailingComma: "es5"`, arrow parens always.

-   Components are **arrow functions** with a `default export`, one per file.
-   Files containing JSX use the **`.jsx`** extension. Plain logic uses `.js`. `src/pages/index.js` exists purely as a Gatsby entry point that re-exports `Home.jsx` — don't put JSX in it.
-   `react/prop-types` is a **warning**, not an error. Adding PropTypes to components you touch is welcome (it's on the roadmap), but don't do a repo-wide migration unless asked.
-   ESLint extends `sonarjs`, `jsx-a11y`, `react-hooks`, and `import` recommended sets. Most rules are warnings; keep new warnings at zero.

## Architecture

### The store is NOT Redux

`src/store/StoreProvider.jsx` is a hand-rolled Context + `useReducer` store that **deliberately mimics the Redux API**:

```js
import { useDispatch, useSelector } from '../store/StoreProvider';
import { selectToggleableJsonResume } from '../store/selectors';
```

`react-redux` is not a dependency. Do not import from it, do not reach for Redux middleware, and do not suggest Redux DevTools wiring as a drop-in (it's a roadmap item requiring real work).

Actions are plain factory functions in `src/store/actions/`, one per file, default-exported. Note they do **not** use a `payload` key — the value is attached under a domain-named key that the reducer reads directly:

```js
const setResumeWork = (work) => ({ type: 'SET_RESUME_WORK', work });
// reducer reads action.work — not action.payload
```

The reducer is a single `switch` in `src/store/reducer/index.js`. To add state: create the action file, add the case to the reducer, add a selector to `selectors.js`.

### The "toggleable" data model

This is the single most important concept in the codebase. Resume data exists in **two shapes**:

1. **Plain JSON Resume** — standard [jsonresume schema](https://github.com/jsonresume/resume-schema) v1.0.0. What users upload and download.
2. **Toggleable** — every field wrapped as `{ enabled: boolean, value: any }`, recursively. This is what lives in the store and drives the sidebar checkboxes that let users hide individual entries.

```jsonc
// plain                    // toggleable
{ "work": [...] }    <->    { "work": { "enabled": true, "value": [...] } }
```

Conversion happens in `src/utils/utils.js`:

-   `convertToToggleableObject(obj, ignoredProperties)` — on import.
-   `convertToRegularObject(obj, ignoredProperties)` — on render/download; disabled fields collapse to an empty default.

Both take an `ignoredProperties` list of keys that stay unwrapped (`coverLetter`, `llmPrompt`, `__translation__`, `enableSourceDataDownload`, `meta`, `$schema`). The two functions have **different defaults** — check which you need.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blopa/Resume-Builder](https://github.com/blopa/Resume-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
