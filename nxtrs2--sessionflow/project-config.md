---
trigger: always_on
description: Guidance for Claude Code (or any AI agent) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (or any AI agent) working in this repository.

## Project Overview

**Session Flow** is a browser-based music player/practice tool built for session
musicians. A musician uploads a "master" mix (MP3) plus optional isolated
instrument stems, and the app renders a scrolling vertical timeline synced to
the audio via [Tone.js](https://tonejs.github.io/). While the track plays, the
timeline shows bar/beat numbers, user-placed **markers** (named positions) and
per-instrument **events** ("prompts" — timed text cues like "Chorus!" or "Come
in here") that count down before they appear. It supports tempo/time-signature
config, a metronome click, count-in/count-out, loop practice between two
markers, and per-instrument volume/mute/solo mixing.

- Live demo (legacy web/Supabase build): https://sessionflow.nxt.rs
- This app is being migrated from a Supabase-backed web app to a **Tauri**
  desktop app with **local filesystem storage** — there is no backend/server
  and no user accounts. All project data (metadata + audio files) lives under
  the OS app-data directory, managed via `src/local/projectStore.ts`. See
  "Local storage model" below. Supabase has been fully removed from the app
  (no `@supabase/supabase-js` dependency, no network calls for project data).

## Tech Stack

- **React 18** + **TypeScript**, built with **Vite 6**
- **Tauri 2** (`src-tauri/`, Rust) — desktop shell; `@tauri-apps/plugin-fs` and
  `@tauri-apps/plugin-dialog` for local filesystem access from the frontend
- **Tone.js** — Web Audio playback, transport/clock, scheduling, synth click
- **lucide-react** — icon set
- Plain CSS (`App.css`, `index.css`, component-level `.css` files) — no CSS
  framework, no CSS-in-JS
- **ESLint 9** flat config (`eslint.config.js`) with `typescript-eslint`,
  `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`

No test framework is configured in this repo (no test runner, no test files).

## Commands

```bash
npm run dev         # start Vite dev server only (browser preview; local fs calls will fail — no Tauri IPC bridge)
npm run tauri dev   # start the actual desktop app (Vite + Rust, hot-reloaded)
npm run build       # tsc -b (project references) + vite build -> dist/
npm run tauri build # produce a distributable desktop app bundle
npm run lint        # eslint .
npm run preview     # preview the production web build locally (same caveat as `npm run dev`)
```

There is no `npm start` script despite what the README says, and no test
command — do not assume either exists.

Building/running the Tauri shell requires a working Rust toolchain (`cargo`,
`rustc`) in addition to Node — install via [rustup](https://rustup.rs) if
missing. First build compiles ~350 Rust crates and can take a couple of
minutes; subsequent builds are incremental.

## Environment Variables

Vite is configured (see `vite.config.ts`) to expose the entire loaded `.env`
as `process.env.*` (not the usual `import.meta.env.VITE_*` pattern). There are
currently no required environment variables for the app to run — the old
`REACT_SUPABASE_URL` / `REACT_SUPABASE_ANON_KEY` vars are vestigial leftovers
from the pre-migration Supabase backend and are no longer read anywhere in
`src/`.

## Architecture

### Provider hierarchy (`src/main.tsx`)

```
initProjectStore()      awaited before the app is rendered (caches appDataDir)
<SessionProvider>        local auth stub — always "logged in", no backend
  <AppProviders>
    <ProjectsProvider>   list of local projects, CRUD via src/local/projectStore.ts
      <CurrentProjectProvider>  in-memory state of the currently loaded project
        <App />
```

- `main.tsx` calls and awaits `initProjectStore()` (`src/local/projectStore.ts`)
  before calling `createRoot(...).render(...)`, so every component can assume
  the local project store is ready and `resolveAssetUrl` can resolve
  synchronously.
- `AppProviders` (`src/contexts/AppProviders.tsx`) no longer threads any
  session/user info into the providers below it — `ProjectsProvider` and
  `CurrentProjectProvider` are self-sufficient now that there's no per-user
  backend.
- Each context has a matching `use*` hook in `src/hooks/` that throws if used
  outside its provider (`useSession`, `useProjects`, `useCurrentProject`).

### State ownership

- **`SessionProvider`** — a local no-op stub (`isLoggedIn: true` always,
  `session: null`, `signIn`/`signUp`/`signOut` are no-ops resolving
  immediately). The interface (`src/contexts/SessionContext.tsx`) is kept
  intact — including its own local `AuthUser`/`AuthSession`/`AuthErrorLike`
  types (no `@supabase/supabase-js` dependency anymore) — so a future
  cloud-sync backend can implement it for real without touching call sites
  like `Login`/`Register`/`Profile`.
- **`ProjectsProvider`** — `projects[]` (from `listProjects()`),
  `currentProject`, demo-loaded flag, and `createProject` / `updateProject` /
  `deleteProject`, all delegating to `src/local/projectStore.ts` for the
  actual filesystem reads/writes.
- **`CurrentProjectProvider`** — the *loaded* project's mutable editing state:
  `instruments[]`, `selectedInstrument`, `projectNeedSave` (dirty flag),
  `updateProjectSongData` (persists the full `SongData` blob back to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nxtrs2/sessionflow](https://github.com/nxtrs2/sessionflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
