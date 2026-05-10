---
trigger: always_on
description: Animated Bitcoin educational explainers using React. Recorded to MP4 via Playwright + FFmpeg, with automated visual QA.
---

# Bitcoin Error Explainer — Animated Video Series

Animated Bitcoin educational explainers using React. Recorded to MP4 via Playwright + FFmpeg, with automated visual QA.

## Workspace
- `client/src/episodes/` — each episode folder has `VideoTemplate.tsx` + custom components
- `client/src/lib/video/` — shared hooks (`useVideoPlayer`), canvas primitives (`CE`, `morph`, `sceneRange`), `DevControls`, animation presets, diagram components
- `scripts/` — auto-episode pipeline (`auto-episode.sh`), **MP4 export (`record.mjs`)**, visual QA (`visual-qa.mjs`), scene screenshots (`screenshot-scenes.mjs`)
- `client/public/audio/` — scene voiceover MP3s
- `references/` — brand guidelines, writing style references

## Role-Specific Guides

This project splits context by agent role to reduce token cost and keep each agent focused:

- **`CLAUDE-build.md`** — Animation toolkit, GSAP utilities, VideoTemplate patterns, episode architecture. For build-phase agents only.
- **`CLAUDE-critic.md`** — Quality bar, sameness checklist, visual distinction rules, episode registry. For critique-phase agents only.
- **`CLAUDE-research.md`** — Teaching approaches, content checklist, tone/voice. For research-phase agents only.

Build agents: read `CLAUDE-build.md`. Critics: read `CLAUDE-critic.md`. Research agents: read `CLAUDE-research.md`. All agents should read this core file.

## Automated Visual QA (`scripts/visual-qa.mjs`)

**Run after building any episode.** Opens the episode in Playwright at 1920×1080, steps through every scene, checks element positions with `getBoundingClientRect()`.

```bash
node scripts/visual-qa.mjs ep11 ./visual-qa-output
```

Reports: **FAIL** (off-screen near-miss), **WARN** (clipped >40% or far off-screen elements). Generates screenshots + markdown report.

**Do NOT write manual POSITION AUDIT comments** — the automated tool replaces manual math audits.

## Scene Rules
- **One idea per scene.** One concept, one step, one point.
- **Scene 1 = Title.** Scene 2 = start from familiar ground — don't open with jargon. A cold open on "Merkle proofs" loses people instantly. Instead: "A block bundles transactions" → "How do you prove yours is inside?" The hook emerges from the progression, not from forcing a dramatic opener. Only use a punchy hook-first opening when the topic is already familiar to the audience.
- **Last scene = CTA** ("Follow @bitcoin_devs") + optional series teaser for next episode.
- **Use as many scenes as needed.** More scenes with less content each > fewer dense scenes.
- **Motivation before mechanism.** Before any scene showing HOW something works (a formula, algorithm, technique), there MUST be a preceding scene establishing WHY — the problem it solves or the question it answers. A viewer should never think "why are we doing this?" If you're about to show math, first show the problem the math solves. Example: don't jump to `7^n mod 15` — first show that "multiplying is easy, un-multiplying is hard, and Bitcoin's security depends on that gap."

## Text Rules
Text and visuals work TOGETHER. The best scenes have **text integrated into the visual** — labels on diagrams, values inside blocks, formulas next to the thing they describe. Think 3Blue1Brown: equations animate alongside the geometry, labels point to the thing they name.

Rules:
- **Visual leads, text clarifies.** Every scene that teaches a concept needs a **real animated visual that demonstrates the mechanism** plus at least one teaching anchor (label, value, caption) that clarifies what the viewer is seeing. The visual does the heavy lifting — text supports it. If a scene is just text panels with entrance animations, the visual isn't leading. Pure unlabeled animation is only OK for title cards, mood beats, or very short transitions.
- **No paragraphs on screen.** 3+ sentences = split across scenes.
- **Use real values.** "bitcoin" → `01100010...` beats "the input gets converted to binary."
- **Progressive reveal.** Each scene adds ONE piece. Like a conversation, not a lecture.
- **Breathing room.** Whitespace is content. Let animations breathe.
- **Questions and quizzes are powerful.** Use when needed, recomended to keep the user hooked.

## Tone & Voice
- Casual-educational, peer-to-peer. ELI5 ethos on deep topics.
- Direct address: "Let's see...", "Now let's look inside..."
- Conversational pacing. Never academic or stiff.
- **Never force analogies.** Only when they map naturally and illuminate the concept.

## Visual Identity

### Color Palette Modes (`--palette` flag)

The `--palette` flag on `auto-episode.sh` controls color constraints:
- **`grayscale`** — black, white, grays only. One accent color allowed for emphasis. Stark, data-focused look.
- **`brand`** — BDP brand palette only (see `references/brand-guidelines.md`). Orange, blue, green, pink, purple + neutrals.
- **`free`** (default) — no restrictions. Pick whatever serves the mood.

Every episode defines its palette in `EP_COLORS` in `constants.ts`. The `--palette` flag guides what goes in it.

### Everything Else Must Vary Per Episode

**Each episode defines its own palette** in `constants.ts`:
```ts
export const EP_COLORS = {
  bg: '#0F172A',          // dark slate — security/attack mood

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitcoin-dev-project/ExplainLab](https://github.com/bitcoin-dev-project/ExplainLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
