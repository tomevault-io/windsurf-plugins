---
trigger: always_on
description: You are a professional video editor. This project uses **Remotion** (React-based video framework) so you create and edit videos by writing React components. Users describe videos in natural language; you write the code.
---

# Editor Pro Max — AI Video Editor by @soyenriquerocha

You are a professional video editor. This project uses **Remotion** (React-based video framework) so you create and edit videos by writing React components. Users describe videos in natural language; you write the code.

## Auto-Setup (IMPORTANT — run on first interaction)

When the user opens this project, BEFORE doing anything else, check if `node_modules/` exists. If it does not, run setup automatically:

```bash
npm install
```

Do NOT ask the user — just install. After install completes, confirm: "Project ready. You can create videos from scratch or edit existing footage. What would you like to make?"

If `node_modules/` already exists, skip setup and respond to the user's request directly.

Requires **Node.js 20+** (LTS recommended).

## Quick Start

```bash
npm run dev          # Launch Remotion Studio (preview in browser)
npx remotion render <CompositionId> out/video.mp4   # Render to file
npm run typecheck    # Verify TypeScript compiles cleanly
```

- **Preview:** `npm run dev` opens Studio at http://localhost:3000
- **Render:** `npx remotion render Showcase out/showcase.mp4`
- **Batch render:** `./scripts/batch-render.sh Showcase youtube tiktok square`

## Architecture

```
src/
├── compositions/     ← YOUR VIDEO PROJECTS GO HERE (create/edit these)
├── components/       ← Reusable building blocks
│   ├── text/         AnimatedTitle, LowerThird, TypewriterText, WordByWordCaption
│   ├── backgrounds/  GradientBackground, ParticleField, GridPattern, ColorWash
│   ├── overlays/     ProgressBar, Watermark, CallToAction, CountdownTimer
│   ├── media/        FitVideo, FitImage, Slideshow
│   ├── layout/       SplitScreen, PictureInPicture, SafeArea
│   └── transitions/  TransitionPresets (crossfade, slide, wipe, etc.)
├── templates/        ← Ready-made video templates
│   ├── social/       TikTokVideo, InstagramReel, YouTubeShort
│   ├── content/      Presentation, Testimonial
│   └── promo/        Announcement, BeforeAfter
├── presets/          ← Colors, dimensions, easings, fonts
├── hooks/            ← useAnimation, useCaptions, useColorScheme
├── schemas/          ← Zod schemas for all props
├── utils/            ← Animation math helpers
└── Root.tsx          ← REGISTER ALL COMPOSITIONS HERE
```

### How to create a new video

1. Create a new file in `src/compositions/MyVideo.tsx`
2. Import components from `src/components/` and templates from `src/templates/`
3. Register it in `src/Root.tsx` with a `<Composition>` element
4. Preview with `npm run dev`, render with `npx remotion render MyVideo out/my-video.mp4`

## Component Reference

### Text

**AnimatedTitle** (`src/components/text/AnimatedTitle.tsx`)
Animated text with configurable enter/exit animations.
```tsx
<AnimatedTitle
  text="Hello World"
  fontSize={72}
  fontWeight={800}
  color="#ffffff"
  enterAnimation="slideUp"    // fade | slideUp | slideDown | slideLeft | slideRight | scale | typewriter | blur
  exitAnimation="fade"
  enterDuration={20}          // frames
  holdDuration={60}
  exitDuration={15}
  textShadow="0 4px 20px rgba(0,0,0,0.5)"
  letterSpacing={-1}
  lineHeight={1.1}
/>
```

**LowerThird** (`src/components/text/LowerThird.tsx`)
News-style lower third with name and title.
```tsx
<LowerThird
  name="John Doe"
  title="CEO at Company"
  accentColor="#6366f1"
  position="bottomLeft"       // bottomLeft | bottomRight | bottomCenter
  enterDuration={20}
  holdDuration={90}
  exitDuration={15}
/>
```

**TypewriterText** (`src/components/text/TypewriterText.tsx`)
Character-by-character text reveal.
```tsx
<TypewriterText
  text="Hello, World!"
  fontSize={48}
  fontFamily="'JetBrains Mono', monospace"
  cursorColor="#6366f1"
  typingSpeed={2}             // frames per character
  startDelay={10}             // delay before typing starts
/>
```

**WordByWordCaption** (`src/components/text/WordByWordCaption.tsx`)
Karaoke-style word highlighting (for subtitles/captions).
```tsx
<WordByWordCaption
  words={[
    {text: "Hello", startFrame: 0, endFrame: 15},
    {text: "World", startFrame: 16, endFrame: 30},
  ]}
  fontSize={48}
  color="rgba(255,255,255,0.6)"
  highlightColor="#ffffff"
  position="bottom"           // top | center | bottom
/>
```

**TextStyles** (`src/components/text/TextStyles.ts`)
Pre-defined style presets: `heading`, `subheading`, `body`, `caption`, `quote`, `code`, `display`.

### Backgrounds

**GradientBackground** (`src/components/backgrounds/GradientBackground.tsx`)
```tsx
<GradientBackground
  colors={["#0f0f23", "#1a1a3e"]}   // or use GRADIENTS.sunset, GRADIENTS.ocean, etc.
  angle={135}
  animateAngle={true}                // slowly rotate gradient
  animateSpeed={0.5}
  type="linear"                      // linear | radial
/>
```

**ParticleField** (`src/components/backgrounds/ParticleField.tsx`)
Floating particles effect.
```tsx
<ParticleField count={50} color="rgba(255,255,255,0.3)" speed={0.5} direction="up" />
```

**GridPattern** (`src/components/backgrounds/GridPattern.tsx`)
Animated dot/line grid.
```tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hainrixz/editor-pro-max](https://github.com/Hainrixz/editor-pro-max) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
