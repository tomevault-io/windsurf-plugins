---
trigger: always_on
description: frame-smith motion skill — never-cheap, effect-engine-driven promo videos as Remotion compositions. A DIRECTOR, not a renderer: reads the brief, sets the register (cinematic landscape vs. vertical short), picks ONE effect engine (poster-cube tumble · capsule theme-flip · architecture data-pulse · particle-burst album), lays a beat timeline, choreographs the reveals, scores it with music + SFX (audio-visual sync), writes the .tsx and registers it. Runs ON TOP OF a video runtime — needs the hyperf
---



# frame-smith — Motion-Crafted · Effect-Engine-Driven · Never Cheap

> **frame-smith is a director, not a renderer.** It decides *what the video is* — register, effect engine, beat timeline, motion craft — and writes a Remotion composition (`.tsx`) that a video runtime renders. It does **not** own the runtime. The runtime is the `hyperframe` skill (the effect-template library, carries its own Remotion) **or** the `remotion` skill (the bare runtime + scaffolding). **frame-smith needs exactly one of them present** and checks first (§0).
>
> frame-smith builds two registers and routes by them (§0.A):
> - **cinematic** — a landscape HyperFrame clip, `1920×1080 · 30fps · 12–24s`, **pure visual, no voiceover**. Optimize for **one spectacular effect engine + a tight beat arc + a wordmark payoff**. This is the register of the four engine archetypes (Poster-cube tumble, Capsule theme-flip, Architecture data-pulse, Particle-burst album).
> - **short** — a vertical short-video, `1080×1920`, for 抖音/小红书/视频号. May carry **subtitles / voiceover**. Optimize for **hook in the first 1.5s + readable-inside-safe-zone + rhythm cut to narration**. Obeys the `S = width/720` scale system and the 抖音 safe zones.
>
> The through-line is identical: **high motion craft, zero motion-slop.** What applies to **both** is the **motion craft floor** (`references/motion-floor.md` — 30fps, the `f(sec)` second→frame helper, `clampOpts`, the export三件套, easing over `linear`, terminal-frame discipline) and the **anti-cheap-motion blacklist** (§6). What **forks** is orientation, safe-zone math, and whether copy is on screen.
>
> Every rule below is **contextual**. Nothing fires automatically. Read the brief, set the register, then pull only the engine that fits. A skill that ships the same fade-up-slide-in clip for every brief has failed — that *is* the slop.

---

## How to use this skill

**0. ENVIRONMENT FIRST — this is a hard gate, not a suggestion.** Before anything else, confirm a video runtime is present. frame-smith writes compositions but cannot render them alone. Load `references/environment.md` and run its check: **at least one of the `hyperframe` or `remotion` skills must be available.**
   - **Neither present** → STOP. Do not write any `.tsx`. Surface the two install options (prefer `hyperframe` — it carries the effect templates you'll lift from; `remotion` is the leaner runtime-only path) and wait. `environment.md` has the exact detection steps and install/bootstrap guidance.
   - **One present** → note which (it decides where the composition file lands and how you register + preview it), then proceed.

1. Run **§0 Brief Read** — infer the **register** (cinematic vs. short) and the **one-line intent** before touching code. Name the lazy default and reject it. Output a short Frame Read, then **stop for confirmation.**
2. Set the **§1 Three Dials** (ENERGY · SPECTACLE · DENSITY). Short register pins DENSITY toward legibility; cinematic pins SPECTACLE high.
3. **Lay the motion craft floor** (`references/motion-floor.md`) — the same for both registers: 30fps, `const f = (s) => Math.round(s*FPS)`, `clampOpts`, easing library, the export三件套 (`Component` / `ComponentCover` / `COMPONENT_FRAMES`). Then the **orientation substrate forks**: cinematic → `1920×1080`, no safe-zone tax, wordmark grammar; short → `1080×1920`, `S = width/720` scale on every pixel, 抖音 safe zones, subtitle rhythm.
4. **Pick ONE effect engine** from `references/effect-catalog.md` — the engine is the video's soul. One, done at 100%, not three at 40%. The catalog is organized by the four engine archetypes plus their reusable sub-effects (3D CSS solids, stadium/pill mask theme-flip, trapezoid data envelopes + counters, SVG stroke-writing + particle burst + 3D ring). Match engine to intent; don't reach for the same one every time.
5. **Lay the beat timeline** (`references/beat-structure.md`) — a video is beats, not a page. Fix `H0…Hn` boundaries in seconds, give each beat one job, and make the last beat *land* (wordmark, logo, CTA). Pick the narrative skeleton for the register.
6. **Choreograph the reveals** (`references/reveals.md`) — how text and elements *enter and exit* is the readable surface of the whole clip. Vary the vector, stagger the siblings, mask the edge, give the hero its own move. The #1 slop is everything fading-up together.
7. **Score it — sound design** (`references/sound-design.md`) — **sound is half the craft, and it is not optional on cinematic either** (no voiceover ≠ no sound). Choose a BGM matched to ENERGY + genre with head/tail fades; place 3–6 SFX to accent the key beats; and **lock audio-visual sync** — every SFX and the BGM drop land on the same beat frame as their picture. This is the highest-leverage "feels professional" move in the skill.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mouse-lin/frame-smith](https://github.com/mouse-lin/frame-smith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
