---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **professional Remotion video component library** for creating tech tutorial videos with Apple-style aesthetics and FilmStorm-level quality.

**Goal**: Build a reusable component library for AI tech content creators, featuring:
- Apple-style minimal design (#000000 background + #007AFF accent)
- Organic, non-mechanical animations
- Professional SVG icons (no emoji)
- Code demonstrations with syntax highlighting
- Data visualization and metrics

---

## Development Commands

```bash
# Start Remotion Studio (live preview at http://localhost:3000)
npm run dev

# Run linting (ESLint + TypeScript)
npm run lint

# Bundle for rendering
npm run build

# Render specific composition
npx remotion render src/index.ts <CompositionID> out/video.mp4

# Upgrade Remotion
npm run upgrade
```

---

## Architecture

### Design System (src/design-system/)

**tokens.ts** - Centralized design constants:
- Colors: Apple-style palette (#000000 bg, #007AFF primary)
- Fonts: SF Pro Display/Text, JetBrains Mono
- Sizes: Large typography (hero: 120px, h1: 88px)
- Spacing, radius, animation durations

**animations.ts** - Animation presets:
- Spring presets: smooth, snappy, bouncy, heavy, playful
- Entrance animations: fade, slideUp, slideLeft, scale
- Decorative: lineExpand, glowSweep, breathe

### Component Library (src/components/new/)

All new components go here. Import via `src/components/new/index.ts`.

#### Core Components

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **HeroTitle** | Opening titles | Reveal animation, glow sweep, tags |
| **SectionTitle** | Chapter headers | Section number, progress bar, left indicator |
| **CodeTerminal** | Code display | macOS terminal style, line-by-line typing, syntax highlight |
| **AnimatedList** | Feature lists | Staggered entrance, icons, check animations |
| **FeatureCard** | Feature showcases | Icon + title + description, grid layout (isFallback) |
| **FeatureGrid** | Feature grid | Grid of FeatureCard components |
| **MetricCard** | Data metrics | Animated number counting, large typography |
| **MetricRow** | Metric row | Horizontal row of MetricCard components |
| **ComparisonCards** | Comparisons | Side-by-side comparison with highlight |
| **ProductIntro** | Product intro | Full-screen product showcase with features |
| **SubtitleOverlay** | Subtitles | Word-level subtitle sync with fade animations |
| **Transitions** | Scene transitions | Fade, Slide, LightSweep, ZoomBlur, CurtainReveal |

#### Data & Charts

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **BarChart** | Bar charts | Animated bars via Recharts, style-aware |
| **LineChart** | Line charts | Progressive reveal, style-aware |
| **PieChart** | Pie charts | Animated slices, style-aware |
| **DataTable** | Tables | Animated rows with zebra striping |
| **HighlightQuote** | Quotes | Highlighted quote with attribution |
| **DataHighlight** | Data callout | Emphasized data point display |

#### Visual & Effects

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **ThreeScene** | 3D scenes | Three.js canvas via @remotion/three |
| **MotionBlurWrapper** | Motion blur | CameraMotionBlur from @remotion/motion-blur |
| **LightLeakOverlay** | Light effects | Cinematic light leak from @remotion/light-leaks |
| **LottieAnimation** | Lottie | Lottie JSON animation playback |
| **CausalGraph** | Causal diagrams | Node-link diagram with animated edges |
| **EvolutionTree** | Evolution | Tree diagram showing version progression |
| **KnowledgeWeb** | Knowledge graph | Interactive web of concepts |
| **ProcessFlow** | Flow diagrams | Step-by-step process visualization |

#### Text Effects

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **TypewriterText** | Typing effect | Character-by-character reveal |
| **TypewriterScene** | Typing scene | Full scene with TypewriterText |
| **CommentBubble** | Comments | Chat/comment bubble with avatar |

#### Icon Library (src/components/new/Icons.tsx)

**50+ SVG icons** based on Heroicons (MIT) and Lucide (ISC) - free for commercial use.

Common icons:
- Tech: Computer, Bot, Terminal, Code, Database, Network
- Actions: Check, Plus, Minus, Close, ArrowRight, Download
- General: User, File, Folder, Calendar, Clock, Settings

**Usage**:
```tsx
import { Zap, Lock, Computer } from '../components/new/Icons';
<Zap size={24} color="#007AFF" strokeWidth={2} />
```

**Important**: Never use emoji. Always use icons from Icons.tsx.

---

## Component Usage Patterns

### HeroTitle
```tsx
<HeroTitle
  title="OpenClaw"
  subtitle="完全新手指南"
  tags={["AI 编码助手", "开源免费", "自托管部署"]}
/>
```

### CodeTerminal
```tsx
<CodeTerminal
  code={`npm install -g openclaw`}
  language="bash"
  filename="install.sh"
  typingSpeed={1}
  showLineNumbers={true}
/>
```

### AnimatedList (with icons)
```tsx
<AnimatedList
  items={[
    { title: '...', description: '...', icon: 'computer' },
    { title: '...', description: '...', icon: 'bot' },
  ]}
  variant="card"
/>
```

### FeatureGrid (with icons)
```tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clear2x/ContentFlow](https://github.com/clear2x/ContentFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
