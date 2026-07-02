---
trigger: always_on
description: > **Default behavior:** After every session, update the `## Session Handoff` section with current progress, decisions made, key variables, and next steps.
---

# CLAUDE.md

> **Default behavior:** After every session, update the `## Session Handoff` section with current progress, decisions made, key variables, and next steps.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server with Turbopack
npm run build    # Production build
npm run lint     # ESLint check
```

No test suite is configured.

## Architecture

**Juoma** is a client-side-only Finnish drinking game app (Next.js 16, React 19, Tailwind CSS). There is no backend, no database, and no persistent state — all game state lives in React component memory.

### Routing — mixed Pages + App Router

The project uses **both** Next.js routers simultaneously:
- `src/pages/` — active Pages Router (`_app.tsx`, `index.tsx`, `play.tsx`, `learn.tsx`, `support.tsx`)
- `src/app/` — App Router used only for the root layout and theme provider

All real page navigation goes through the Pages Router. The App Router layout wraps it via `ThemeProvider`. **`src/pages/index.tsx` is now the primary entry point** — it handles player setup, game selection, and inline game rendering. `/play` still exists but is no longer the main flow.

### Game system

Games live in `src/components/games/`. Each game component receives `{ players: string[], onBack: () => void }` and is responsible for its own gameplay logic. Games are wired into `src/pages/index.tsx` via the `GAMES` array.

**Implementation status:**
- `Viinapiru.tsx` — fully implemented (song playback + random starter picker)
- `PullonPyoritys.tsx` — fully implemented (bottle spin, history tracking, sip counter)
- `Placeholder.tsx` — partial (random task + sip assignment from task pool, no win condition)
- `Malte.tsx` — fully implemented (see Malte section below)
- `Hitler.tsx` — fully implemented (see Hitler section below)

### Game content

`src/data/tasks.ts` exports `extraTasks`: 245 Finnish-language drinking challenges used by `Placeholder.tsx`.

### Theming

Dark mode is the **default** (localStorage falls back to dark, not light). `src/app/theme/ThemeContext.tsx` manages dark/light mode via a `data-theme` attribute on `<html>`. CSS custom properties (`--background`, `--foreground`, etc.) are in `src/app/globals.css`. Tailwind dark mode strategy is `"class"`.

### Animated background

`src/components/AnimatedBackground.tsx` renders a fixed-position layer of floating amber spark particles (Web Audio API CSS keyframe animations). The body always has an animated dark gradient (`-45deg`, navy/purple/dark blue tones, 20s cycle). This is mounted in `src/layouts/MainLayout.tsx`. Game component outer containers are **transparent** (no `bg-gray-900`) so the gradient shows through; individual cards use `bg-gray-800`.

### Audio

- Viinapiru: MP3 files in `public/audio/` (song1–4.mp3), `<audio>` elements
- Malte: Web Audio API via `src/components/games/malteSounds.ts` — no external files

### Path alias

`@/*` maps to `src/*` (configured in `tsconfig.json`).

---

## Malte — Full Feature Reference

`src/components/games/Malte.tsx` — 4-round card prediction game with bussi endgame.

### Settings (configurable before starting)
| Setting | Default | Notes |
|---|---|---|
| `sipsPerRound` | `[1,2,3,4]` | Sips awarded per round |
| `endgameScaling` | `"default"` | Bussi sip scaling: default (+2/card), double, custom |
| `customStart` | `2` | Starting value for custom bussi scaling |
| `extraRounds` | `true` | Extra rounds for small groups |
| `toastTimer.enabled` | `true` | Auto-dismiss penalty toast |
| `toastTimer.seconds` | `5` | 1–10s slider |
| `sipCounter.enabled` | `true` | Track and display sips per player |
| `sipCounter.startAt` | `20` | First sip-warning threshold |
| `sipCounter.every` | `10` | Repeat warning every N sips after startAt |
| `sounds.enabled` | `true` | Web Audio API sounds |
| `sounds.pack` | `"normal"` | `"normal"` / `"funny"` / `"intense"` |

### Sound packs (`malteSounds.ts`)
- **Normaali**: 3 liquid glugs on drink; soft double-chime on sip warning; 95 BPM kick/snare loop in bussi
- **Hassu**: cartoon pitch-swoosh on drink; wah-wah-wah on warning; 118 BPM syncopated loop
- **Intensiivi**: distorted bass chug on drink; triple air horn on warning; 145 BPM double-kick loop

### Sip counter (Hörppy-laskuri)
- Shown at the bottom of rounds phase and bussi phase
- Warnings fire at `startAt`, `startAt + every`, `startAt + 2*every`, …
- Warning popup is queued behind the current drink toast (appears after toast clears)

### Round 1 guessing rule
- Max 2 guesses. First wrong → show hint (pienempi/suurempi). Second wrong → turn ends automatically (no give-up button).

### Summary (Yhteenveto)
- Players ranked by total sips with 🥇🥈🥉 medals
- Winner row highlighted in amber
- Cards held shown below ranking

---

## Hitler — Full Feature Reference

`src/components/games/Hitler.tsx` — classic Finnish card drinking game.

### Packages
Currently one package: **Perus** (default). Infrastructure exists for multiple packages (add to `PACKAGES` array in the file).

### Card effects (Perus package)
| Card | Effect |
|---|---|
| A | Vesiputous — waterfall |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leotamminen/juoma](https://github.com/leotamminen/juoma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
