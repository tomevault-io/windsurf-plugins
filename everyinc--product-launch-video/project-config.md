---
trigger: always_on
description: >
---


# Product Launch Video Workflow

A repeatable workflow for producing 15-30s product launch videos using Remotion. Built for feature launches, app demos, and before/after transformations destined for X, LinkedIn, YouTube Shorts, and landing pages.

## When to use

- Product launch needs a short, social-ready video
- New feature demo for X/LinkedIn
- Before/after product transformation clip
- Landing page hero video

## Prerequisites

- Node.js 18+
- This repo cloned and `npm install` run
- ffmpeg (only if exporting GIFs)
- A screen recording of the product (30-60s, used as design inspiration only)
- The launch copy: headline, tagline, key features, tone direction

## Workflow

### 1. Gather inputs (parallel)

Pull these in parallel before writing anything:

- **The brief** — launch post, landing page, one-pager. Extract: headline, taglines, key features, tone.
- **Brand tokens** — fonts, colors, icons, logos. If you have a design system, surface its values. Otherwise, pull from the live product.
- **Screen recording** — a 30-60s recording of the product in real use. This is your **design inspiration** and a source of truth for actual UI copy. Never invent labels, agent names, or features that don't exist in the real app.
- **Voice calibration** (optional) — if the video has narration or you're ghostwriting captions in a specific voice, pull recent posts/clips to match cadence and word choice.

### 2. Storyboard before any code

Write a shot-list table. Resist the urge to start coding.

| Beat | Time | Visual | Text overlay |
|------|------|--------|-------------|

**Rules that save you from re-rendering:**

- **Use real product UI copy.** Screenshot the actual app and mirror what's there. Invented labels are the #1 reason launches feel fake.
- **Lead with the viewer's problem** (hook), not the product name.
- **Product must appear within 3 seconds** (a16z Speedrun research on launch videos).
- **Before/after is the money shot** — design the whole video around it.
- **All text must be readable at 720p** (phone feed). Minimum 18px equivalent.
- **85%+ of social views are muted** — burn all captions in. No VO unless explicitly requested.

### 3. Build in Remotion

This repo ships with a minimal Remotion scaffold. Start from `src/LaunchVideo.tsx` and customize:

- Keep the composition thin — extract scenes, theme, fonts, and mock data into their own files as complexity grows
- Register new compositions in `src/Root.tsx` (one per aspect ratio if shipping to multiple platforms)
- Use `staticFile()` for assets in `public/`
- Defaults are 1920×1080 at 30fps, h264 crf 18 — see the Ship section for platform-specific overrides

**Font loading pattern** (prevents character clipping and layout thrash):

```tsx
const [handle] = useState(() => delayRender('fonts'));
useEffect(() => {
  const font = new FontFace('Name', `url(${staticFile('fonts/file.otf')})`);
  font.load().then(f => { document.fonts.add(f); continueRender(handle); });
}, [handle]);
```

**Crossfade transitions** — handle at the orchestrator/root level, not per-scene. Per-scene exit fades fight the orchestrator and cause double-fading.

### 4. Render-audit loop

```bash
# Single frame audit
npx remotion still <comp-id> out/test.png --frame 60

# Full render
npx remotion render <comp-id> out/video.mp4 --codec=h264 --crf=18
```

Render one frame per scene first. Fix layout/clipping issues before full render. Check:

- Text not clipped (line-height >= 1.2 for serif fonts with descenders)
- Panels not stacking unexpectedly (AbsoluteFill defaults to `flex-direction: column`)
- Content not truncated in bubbles, cards, or chat UI

### 5. Multi-agent review (the step that catches everything)

After 2-3 self-iterations, render ~20 frames (peak moments + transition frames) to `out/review/` and spawn **4 review agents in parallel**. Each reads the PNG frames and returns a prioritized fix list.

1. **Design/layout critic** — off-kilter elements, misalignment, proportions, cropping
2. **Text/readability critic** — legibility at 720p, contrast, truncation, typos
3. **Narrative/pacing critic** — story arc, attention peaks/drops, earned duration, CTA strength
4. **Brand consistency critic** — fonts, colors, UI chrome matching the real product

Compile all 4 critic outputs + your own notes into a single revision plan before the next render pass. This step finds things you stop seeing after 50 renders.

### 6. Iterate on feedback

Watch the render in QuickTime (or your player of choice) and iterate. Common feedback patterns and their fixes:

- **"This text is cut off"** → line-height or overflow issue
- **"This feels clunky"** → animation timing too fast/slow; check counter digit-flicker rate
- **"You invented this label"** → replace with real UI copy from the screen recording
- **"Hold longer here"** → extend scene duration, keep animation timing the same
- **"A little long overall"** → cut redundant scenes (the narrative critic usually identifies which)

### 7. Ship for the right platform

**Ask the user which platforms they're publishing to before rendering.** Aspect ratio and duration limits differ — a 16:9 clip cropped to 9:16 in post usually looks bad. If the video is going to multiple platforms with different aspect ratios, create a separate Remotion composition for each and render independently.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EveryInc/product-launch-video](https://github.com/EveryInc/product-launch-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
