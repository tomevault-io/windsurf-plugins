---
trigger: always_on
description: ASIST is an Electron desktop application for real-time voice interaction, built with TypeScript,
---

# AGENTS.md

## Project

ASIST is an Electron desktop application for real-time voice interaction, built with TypeScript,
React, electron-vite and Vitest. The documentation for users is on the website
(`website/src/content/docs/`, in Japanese and English), and `docs/development.md` covers running from
source, building, CI and manual validation. Read the relevant implementation and tests before changing
behavior.

Do not read or use `frontend-skill` in this product.

## Architecture

- `src/main/` owns the Electron main-process integrations, services, persistence, sidecars and
  agent execution.
- `src/preload/` exposes the safe bridge to the renderer.
- `src/renderer/` owns the React UI, voice capture, panels and client-side state.
- `src/shared/` holds cross-process contracts and pure, testable logic. `src/shared/ipc.ts` is the
  canonical main/preload/renderer API contract.
- What the OS and the machine can run is decided once in main (`src/main/services/platform.ts`) and
  passed on as capabilities; the renderer and the model's tools read those and never check the OS
  themselves.

Keep process boundaries explicit: renderer code does not reach Node or Electron APIs for an
operation that belongs behind preload or the main process.

## Code

- Persist each fact in one authoritative place and derive secondary views instead of
  synchronizing copies.
- Do not add fallback behavior; fail loudly rather than degrade silently.
- Fix a defect where its cause is, in a form in which it cannot happen, rather than with a guard for the
  one case that showed it; the code after the fix reads better than before. When a fix is much larger
  than the defect, or needs a choice only the user can make, stop and ask instead of patching.
- Code never splits a path or tests it as text with `/`, because Windows writes `C:\` and `\`: main
  uses `node:path`, and shared and renderer code, which cannot import it, use `src/shared/file-path.ts`.
- A JSON file under userData carries the version of its form (`StoredFormat`). Any change to the form,
  an added field included, raises the version, adds an upgrade from the previous version, and adds a
  sample of the new version to `tests/fixtures/stored/`. Upgrades are never removed.
- Extract code only when it creates a coherent responsibility, a reusable boundary or an
  independently testable unit. Introduce a shared abstraction only after two current
  implementations show the same responsibility with meaningful variation.
- A file approaching 500 lines calls for a review of its responsibilities; split it when a coherent
  one can be extracted, not to meet a line count.
- Preserve the approval gate for agent jobs that can write or mutate external state.
- Colours, surfaces, radii and the background come from the theme tokens in
  `src/renderer/src/assets/themes.css`; a component or its CSS never writes a colour itself.
- Keep generated output and local secrets out of Git: never commit `.env`, `node_modules/`, `out/`,
  `dist/`, `coverage/`, logs or TypeScript build info.
- Start every child process with `windowsHide: true`; without it, each one ASIST starts on Windows opens
  a console window.

## Comments

- Write comments in English, in full sentences and the present tense. Japanese appears only as
  data, quoted verbatim.
- Say only what the code cannot: an API quirk and the failure it causes, a measured value with its
  condition and date, a constraint, or why the simpler approach was rejected. Do not restate
  names, narrate steps, or mention history, TODOs, docs, tickets or conversations.
- `/** … */` on exported and non-obvious module-level declarations, without `@param` or `@returns`.
  A file header, if any, goes after the imports.
- Inside a function, `//` on its own line above the code. No trailing comments and no banner
  comments (`/* ---- section ---- */`).
- The same rules apply to CSS and scripts.

## Tests

- Add or update Vitest coverage for behavioral changes, especially shared logic, voice control,
  prompting, retries and IPC contracts.
- A failing test points to a defect. A test does not fail on a deliberate change of wording, a name
  or a configured value, and does not only assert that something exists or is gone.
- Tests protect current behavior or a safety boundary, not the shape of removed code.
- Test names are English. Japanese in a test is fixture data (utterances, note bodies); the text of
  the screen is read through the dictionary (`t('key')`).

## Text

The app runs in eleven locales. These rules hold everywhere; the `ui-text` skill has the rest.

- Text for the screen lives in the dictionary (`src/shared/i18n/messages/`), in all eleven
  languages, written in the same change. No word goes straight into a component, in any language.
- An error is thrown as `errorText(key, values)`, never as a sentence.
- Text for the model is a `PromptText` `{ ja, en }` next to the code, not in the dictionary.

## Decisions

`docs/adr/` keeps the decisions the code cannot show, one file each, and the file names say which
behavior each covers. Before changing how a feature behaves, list the folder and read only the
records whose names cover that behavior; if the change contradicts one, say so to the user first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyosegawa/asist](https://github.com/nyosegawa/asist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
