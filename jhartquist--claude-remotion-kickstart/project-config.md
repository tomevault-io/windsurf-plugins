---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Remotion video project designed to make it easier to create videos programmatically with Claude. The project uses React, TypeScript, and Tailwind CSS.

## Package Manager

**Always use `pnpm` for all commands.** Do not use `npm` or `npx`.

## Common Commands

```bash
# Install dependencies
pnpm i

# Start development preview
pnpm run dev

# Lint code
pnpm run lint

# Render a composition (by ID)
pnpm exec remotion render Example1-Landscape

# Render still image
pnpm exec remotion still Example1-Landscape

# Upgrade Remotion
pnpm run upgrade
```

## Architecture

### Project Structure

```
src/
├── components/              # Reusable components (black/white theme, className override)
│   ├── TitleSlide.tsx       # Full-screen title
│   ├── ContentSlide.tsx     # Header + body text
│   ├── CodeSlide.tsx        # Code with title
│   ├── DiagramSlide.tsx     # Mermaid/D2 diagrams
│   ├── VideoSlide.tsx       # Video playback
│   ├── BRollVideo.tsx       # B-roll with zoom
│   ├── ZoomableVideo.tsx    # Video with zoom segments
│   ├── Screenshot.tsx       # Scrolling screenshot
│   ├── Logo.tsx             # Logo overlay (animated)
│   ├── Caption.tsx          # Subtitle/caption overlay
│   ├── AsciiPlayer.tsx      # Terminal recording playback
│   ├── Code.tsx             # Syntax-highlighted code
│   ├── Diagram.tsx          # Diagram renderer
│   └── Music.tsx            # Background music with fade
├── compositions/
│   ├── example1/            # Reference: basic slideshow
│   └── example2/            # Reference: multi-feature demo
├── utils/
│   ├── createComposition.tsx  # Helper to create compositions
│   └── segmentTranscript.ts   # Parse transcripts for timing
├── config.ts                # Timing utilities: secondsToFrames(), framesToSeconds()
├── presets.ts               # VIDEO_PRESETS (aspect ratios, 60fps)
├── content.ts               # Sample content for component previews
└── Root.tsx                 # Composition registry
```

### Quick Start

1. **Run `/new-composition my-video`** - Create a new composition with boilerplate
2. **Edit `src/compositions/my-video/content.ts`** - Change the text
3. **Edit `src/compositions/my-video/config.ts`** - Adjust timing (in seconds)
4. **Run `pnpm exec remotion render MyVideo`** - Render your video

### Key Concepts

**Components** have black/white defaults with `className` prop for theming:

```tsx
// Default black background, white text
<TitleSlide title="Hello" />

// Custom theme via Tailwind classes
<TitleSlide title="Hello" className="bg-blue-900 text-yellow-300" />
```

**Transitions** use Remotion's built-in `<TransitionSeries>`, not component props:

```tsx
// GOOD: Use Remotion's TransitionSeries for fades
<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={180}>
    <TitleSlide title="Hello" />
  </TransitionSeries.Sequence>
  <TransitionSeries.Transition
    presentation={fade()}
    timing={linearTiming({ durationInFrames: 30 })}
  />
  <TransitionSeries.Sequence durationInFrames={300}>
    <ContentSlide header="Main" content="..." />
  </TransitionSeries.Sequence>
</TransitionSeries>
```

**Timing** uses `<Sequence>` for positioning, not component props:

```tsx
import { secondsToFrames } from "../../config";

// GOOD: Position with Sequence (secondsToFrames defaults to 60fps)
<Sequence from={secondsToFrames(5.2)} durationInFrames={secondsToFrames(2.7)}>
  <Logo src="logo.svg" />
</Sequence>
```

### Timing Utilities

All utilities default to **60fps** and can be used anywhere (components, config files, etc.):

```tsx
import { secondsToFrames, framesToSeconds } from "../../config";

// Convert seconds to frames (defaults to 60fps)
secondsToFrames(2.5)        // => 150 frames
secondsToFrames(2.5, 30)    // => 75 frames (custom fps)

// Convert frames to seconds
framesToSeconds(150)        // => 2.5 seconds
framesToSeconds(150, 30)    // => 5 seconds (custom fps)
```

Use these for transcript-based timing:

```tsx
// In config.ts - define segment timing from transcript
export const SEGMENTS = {
  intro: { start: 0, end: 3.2 },
  feature: { start: 3.2, end: 8.5 },
};

// In Composition.tsx
<Sequence from={secondsToFrames(SEGMENTS.intro.start)}>
  <IntroSegment />
</Sequence>
```

### Composition Pattern

Use the `createComposition` helper:

```typescript
import { createComposition } from "../../utils/createComposition";

const MyVideoComposition: React.FC = () => {
  // ... video content
};

export const MyVideo = createComposition({
  name: "MyVideo",
  component: MyVideoComposition,
  durationInSeconds: 10,
  preset: "Landscape-1080p",
});
```

### Root.tsx Structure

- `Examples` folder with reference compositions
- `Components` folder with previews for each component
- Use `/new-composition` to add your own compositions

### Video Presets

All videos run at **60fps**. Available in `src/presets.ts`:

- `Landscape-720p`: 1280x720 @ 60fps
- `Landscape-1080p`: 1920x1080 @ 60fps
- `Square-1080p`: 1080x1080 @ 60fps
- `Portrait-1080p`: 1080x1920 @ 60fps

### Styling

- Use Tailwind CSS classes
- Default theme: `bg-black text-white`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhartquist/claude-remotion-kickstart](https://github.com/jhartquist/claude-remotion-kickstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
