---
trigger: always_on
description: A cozy, quantitative **voice-feminization training tracker** for Rachel. She records
---

# Voice Garden 🌷 — project guide for Claude

A cozy, quantitative **voice-feminization training tracker** for Rachel. She records
herself (usually the Rainbow Passage), tells you what she was practicing, and you
analyze it and surface — kindly and specifically — what to work on next.

If she shares a recording or asks how a take went, use the **`analyze-voice` skill**
(`.claude/skills/analyze-voice/SKILL.md`). It's the operating manual for the whole
workflow; this file is the why and the shape.

---

## Initial setup (first run — read this first if the project is fresh)

This may be a **starter copy**: the app, the `analyze-voice` skill, the reusable
annotation lib, the shared reference voices (`reference.json`), and an `_example`
annotation template are all here — but there may be **no recordings yet** (empty
`recordings.json`). The user adds their own; everything else is structure for you
to drive.

**Prerequisites** (install whatever's missing):
- **ffmpeg** — `brew install ffmpeg` (macOS) / `apt-get install ffmpeg` (Linux). Required by `analyze.py` to read mp3/m4a.
- **uv** — the Python/dependency manager (https://docs.astral.sh/uv). **Use uv, never pip.**
- **Node + npm** — for the dashboard.

**Bootstrap:**
```fish
uv sync                                   # Python deps (parselmouth, numpy) → .venv
cd dashboard-react && npm install         # dashboard deps
npm run dev                               # → http://localhost:5173
```

**Add the first recording** (this drives the whole dashboard):
```fish
uv run analyze.py "/path/to/recording.mp3" --label "what they were trying"
```
Refresh the dashboard. Then author that take's insight by following the
`analyze-voice` skill — copy `dashboard-react/src/annotations/entries/_example.tsx`
to `00N.tsx` (matching the new recording's id) and fill in the slots.

**Ownership note:** this guide and the skill were written for the original user
("Rachel"). Treat "the user" as whoever you're working with now — keep the same
warm, *compass-not-judge* tone and all the conventions below.

If this is your first time running and it is an empty codebase, please use this info provided to give the user an overview of how to operate this codebase. Be supportive and give them a good experience, I trust you Claude <3
~ Rachel

---

## The vibe (this matters as much as the code)

The aesthetic is **"Animal Crossing but girliepop"**: pastel pink/purple, soft rounded
cards, gentle gradients, comforting — *cute but not over-the-top*. The emotional design
goal is just as important as the metrics:

- **Numbers are a compass, not a judge.** Every metric is framed as a hint toward a goal,
  never a verdict. The footer literally says so. Honor that everywhere.
- **Honest but kind.** Don't inflate progress (if "melody" is really register crashes,
  say so) — but always pair a weakness with a concrete, doable next step, and end warm.
- **Woven-in, personal encouragement.** The little notes ("you're in the *fem* zone —
  165 Hz+ reads feminine to most ears 💕") are a core feature, not decoration. Personalize
  them per recording where it helps (see the annotation slots below).
- Emoji in moderation (🌸🎀🎯💕🌱). Rounded font (`ui-rounded`). Keep it soft.

### Color convention — STRICT
Defined in `dashboard-react/src/zones.ts`:
- `MASC` blue `#bcd3f0` (and `#5e7fb8` strokes) = **masculine / fell-out-of-register ONLY.**
  Never use blue for a generic "bad / needs work." A register crash *is* masculine
  register, so blue is correct there.
- `FEM` pink `#ffb6d5` = good / feminine end. `BUTTER` `#ffe9a8` = neutral / mid.
- `GROW` `#cdc6da` = neutral "room to grow" for **non-gendered** skill gaps (breathy,
  rough, monotone). Use this, not blue, for those.

---

## Iconography & visual motifs

The app has a **custom hand-drawn icon set** (no more raw emoji for structural UI) plus a
**tulip favicon**. When touching the UI or adding icons, stay inside this system.

### The tulip mark 🌷 (favicon + hero)
The mascot is a soft tulip: pink bloom (gradient `#ffc8df→#ff92be`, center petal `#ffb6d5`,
seams `#ff89bb`, a white highlight ellipse) on a green stem/leaves (gradient
`#a6e0bd→#79c797`). The **favicon** wraps it in an iOS-style squircle over the brand gradient
`#ffd6ea→#d3c4ff`. Files live in `dashboard-react/public/`: `favicon.svg` (scalable primary),
`favicon-16/32.png`, `favicon.ico` (multi-res), `apple-touch-icon.png` (180² **full-bleed** —
iOS masks its own corners). Wired in `index.html` with `<link>`s + `theme-color #ffd6ea`. The
same flower (no square, viewBox cropped tight to the bloom) is the hero `<TulipIcon>`.

### The icon set — `src/components/icons.tsx`
Self-contained **inline-SVG React components**, one per section heading, replacing the old
leading emoji. Exports + where they're used (in `App.tsx`):
`TulipIcon` (hero), `BowIcon` (Latest take), `SparkleIcon` (Resonance),
`ContourIcon` (Register & phrasing), `InsightIcon` (Insights), `TrendsIcon` (Trends),
`CardsIcon` (All recordings), `BulbIcon` (What do these mean?).

How to use: `<BowIcon />` — drop it as the first child of a `.section-title` (already
`display:flex; gap:8px; align-items:center`). Props: `{ size?, className?, title? }`. Size

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scratchyone/voice-training-ui](https://github.com/scratchyone/voice-training-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
