---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git workflow

After every meaningful change (new feature, bug fix, refactor), commit and push immediately:

```bash
git add <specific files>
git commit -m "feat|fix|refactor|docs: short description"
git push
```

Use conventional commit prefixes: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`. Never batch unrelated changes into a single commit. Always push after committing so GitHub stays up to date.

## Commands

```bash
npm run dev          # Start dev server (http://localhost:5173/app-familiale-routines/)
npm run build        # TypeScript check + Vite production build
npm test             # Vitest characterization suite (useAppState: migrations, rewards, lifecycle)
npm run sync-assets  # Copy images/music from source folders → public/, regenerate manifests
```

The Vitest suite in `src/hooks/useAppState.test.ts` is the contract for state logic — run it after any change to `useAppState.ts`, `rewardImages.ts` or the migration chain. `npm run build` (tsc) is the second gate.

## Architecture

**Stack**: React 18, TypeScript, Vite 6, Tailwind CSS 3, PWA (vite-plugin-pwa / Workbox), Vitest

**State management**: Single hook `src/hooks/useAppState.ts` — all app state lives here, persisted to `localStorage` via `useLocalStorage`. No external state library.

**Design system**: warm tokens in `tailwind.config.js` (`warm`, `ink`, `line`, `success`, `honey`, `danger`, `night` + `shadow-card/raised/overlay` + `z-overlay/modal/toast`), brand fonts self-hosted via `@fontsource-variable/fredoka` (display) and `@fontsource-variable/nunito` (body), UI primitives in `src/components/ui/` (Card, Button, Pill, Badge, Overlay, ScreenHeader, IconButton, TextInput, FieldLabel). Child identity colors live in `src/theme.ts` (`COLOR_PALETTE`, `tint()` for translucent surfaces, `childTextColor()` for AA-readable colored text). Never concatenate hex + alpha (`color + '15'`) — use `tint()`.

**Screens** (rendered by `src/App.tsx` based on `currentScreen`):
- `home` → `HomeScreen` — routine launcher + timer + parent-gated creation (custom routine form + "Mission express") + universe-unlock banner + "all routines done → new day" banner. Parent access and reward-bearing creation share one **long-press (2 s) gate** (`startLongPress`), with the optional 4-digit `ParentGate.tsx` on top when a PIN is set
- `routine` → `ActiveRoutineScreen` — split-screen (2 children side by side), sounds, music, timers, universe-unlock choice after celebration
- `parent` → `ParentPanel` — reset/stop controls, timer launcher, sanctions, universe access, optional 4-digit parent PIN (`PinSetupOverlay.tsx`/`PinPad.tsx`)
- `gallery` → `GalleryScreen` — per-child reward image collection, **one section per owned universe** on a single scrollable page, shows progress to next universe unlock
- `universe-select` → `UniverseSelectScreen` — per-child universes (parent side): switch among owned, grant locked ones early

**Onboarding** (`OnboardingScreen.tsx`, when `onboardingCompleted` is false): welcome → children (photo strongly suggested, color picked with the child) → starting universe per child → routines. A static splash in `index.html` (`#splash`) shows before React mounts; `App.tsx` fades it out. Logo source: `public/icons/icon.svg` (`AppLogo.tsx` inline copy; `node scripts/generate-icons.mjs` regenerates the PNG icons).

**Universe system (schema V7)**: each child has `universeId` (active pool key in `rewardManifest.ts`) and `unlockedUniverseIds` (owned universes). Universe metadata lives in `src/data/universes.ts`. Pool resolution: `getRewardImagesForChildEntry()` in `src/data/rewardImages.ts` (falls back to legacy index round-robin). Per-universe progress = intersection `unlockedImages ∩ pool`, so switching universes loses nothing. Rewards draw from the union of all owned universes; a cycle reset clears every owned universe's ids once their combined pool is exhausted (see Reward system). To add a universe: drop an image folder in `images_rewards/<Name>/` (or generate one via `node scripts/generate-universe-images.mjs <id>` — reads `GEMINI_API_KEY` from the local vault `C:/secrets/keys.env` first, env var as fallback; never committed), run `npm run sync-assets`, add an entry in `universes.ts` (id = slugified folder name).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thibaultmathieu/app-familiale-routines](https://github.com/thibaultmathieu/app-familiale-routines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
