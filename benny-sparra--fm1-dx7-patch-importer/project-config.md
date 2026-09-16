---
trigger: always_on
description: manages DX7-compatible voices in browser storage and communicates with the hardware through Web
---

# Repository guidance

This file applies to the entire repository.

## Project overview

This is a client-only React and TypeScript editor/librarian for the M-VAVE FM1 synthesiser. It
manages DX7-compatible voices in browser storage and communicates with the hardware through Web
MIDI. There is no application server and no supported device-to-browser bank readback; the browser
library is the source of truth.

Use Node.js 24.18.0 and npm 11.16.0, as pinned by `.node-version` and `package.json`.

## Technology and architecture

- React 19, TypeScript, and Vite provide the application shell and production build.
- Tailwind CSS 4 is configured through the Vite plugin; shared theme and component styles live in
  `src/index.css` and `src/fonts.css`.
- IndexedDB persistence is isolated behind the patch-library storage modules. React hooks own the
  browser-facing orchestration; keep MIDI, storage, and file-format rules in testable `src/lib/`
  modules rather than UI components.
- `i18next` and `react-i18next` provide localisation. `dnd-kit` provides patch reordering, and
  Testing Library with Vitest covers observable UI behaviour.
- Production hosting is static. Cloudflare Pages headers provide the deployed security policy;
  Umami supplies privacy-limited analytics, and Sentry is loaded as an optional monitoring chunk.

## Repository layout

- `src/components/`: UI grouped by editor, MIDI, patch-library, and shared UI concerns.
- `src/hooks/`: React state and browser-integration hooks.
- `src/lib/`: domain logic, MIDI encoding, persistence, and reusable utilities.
- `src/routes/`: the eager root/librarian views and lazy patch editor.
- `src/i18n/locales/`: complete resources for each supported locale.
- `src/data/`: bundled static data.
- `src/assets/`: source artwork; `src/assets/generated/` contains committed responsive derivatives.
- `src/test/`: shared accessibility helpers and rendered accessibility coverage.
- `scripts/`: deterministic repository checks that do not belong in application code.
- `public/`: files served unchanged by Vite.

Use the `@/` alias for cross-directory imports. Use relative imports for a module's immediate local
files when that is clearer.

## Code style

- Follow `.editorconfig` and Prettier: two spaces, single quotes, no semicolons, trailing commas,
  100-column width, and LF endings.
- Let `prettier-plugin-tailwindcss` order utility classes. Add conditional class names through
  `cn(...)`; review formatting changes to conditional strings carefully.
- Prefer small named functions and explicit domain types. Keep state near the behavior that owns it.
- Keep TypeScript compatible with `verbatimModuleSyntax` and `erasableSyntaxOnly`; use type-only
  imports where required and avoid runtime TypeScript-only constructs.
- Use Lucide icons and the existing components in `src/components/ui/` before adding new UI
  primitives.
- Do not use dangerous lint autofixes. `npm run lint:fix` is the supported autofix command.
- Do not throw from React state updater functions. React runs them during render, so the caller's
  `try/catch` never sees the error. Work out the next state where the caller can catch a failure,
  then set it.
- Keep one source for shared constants and helpers such as key lists, limits, and value formatting.
  Reuse or export the existing one rather than copying it into another module.

## Behavioral constraints

### Persistence

- A storage read failure must never silently create and save factory data over a user's workspace.
- Treat a missing workspace differently from an unreadable or incompatible workspace.
- Keep session-only recovery explicit, preserve unsaved in-memory data after write failures, and
  serialize saves so an older snapshot cannot become final storage.
- Cancellation, retry, disposal, and completions arriving after unmount are normal cases and require
  deterministic handling and tests.
- Debounced saves must not lose recent edits: write any pending save immediately when the page is
  hidden or closed, and warn before leaving while a save has not committed.
- Read and write `localStorage` and `sessionStorage` only inside `try/catch`. Blocked or throwing
  storage must leave the feature working with a safe default, never break the action that uses it.

### Legacy stored data compatibility

Users keep their only copy of their voices in browser storage, so every release must be able to
open everything an earlier release could have saved.

- Treat every persisted shape as a public format: the IndexedDB database name, schema version,
  object store names, key paths, and record keys in `src/lib/patch-library-storage.ts`; the
  versioned workspace record (`StoredPatchLibrary`) and saved bank (`NamedBank`) shapes; and
  `localStorage` keys such as `fm1-language`, `fm1-colourway`, and the MIDI port and help-dialog
  keys. Do not rename, remove, or repurpose any of them.
- Changing a stored shape means bumping its record `version` and adding an upgrade path that reads
  every earlier version. Never drop support for an old version, and never reuse a version number for
  a different shape.
- Upgrade on read, in memory. Write back only in the newest format through the normal save path,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benny-sparra/fm1-dx7-patch-importer](https://github.com/benny-sparra/fm1-dx7-patch-importer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
