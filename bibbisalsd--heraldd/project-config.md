---
trigger: always_on
description: > **For**: Gemini CLI agent sessions
---

# Harold / Jarvis Core - Developer Handoff for Gemini CLI

> **For**: Gemini CLI agent sessions
> **Primary task**: Keep the Herald website up to date with the current Jarvis Core codebase state
> **Last updated**: 2026-04-08

## Quick Orientation

This is **Jarvis Core** (codename Harold) — a local-first AI assistant runtime built in Python. It runs entirely on the user's machine using Ollama for LLM inference, Kokoro ONNX for TTS, and faster-whisper for STT. No cloud APIs required.

The project is a merger of two codebases:
- **Esoteric v0.2** (the "Mind") — cognitive layer: world model, evidence store, belief state, judge, CRSIS self-improvement
- **CLLM** (the "Spinal Cord") — deterministic pipeline: routing cascade, tool policy, admission control, contracts, rendering

**Owner**: billybart (James)
**Hardware**: i5-13420H, 16GB RAM, RTX 4050 6GB VRAM, Pop!_OS Linux

---

## Project Paths

| What | Path |
|------|------|
| Jarvis Core (Python) | `/home/billybart/Downloads/Harold/esotericv0.2CURRENT/Harold/Jarvis/Jarviscore/` |
| Herald Website (Next.js) | `/home/billybart/Downloads/Harold/site/New folder/` |
| Detailed handoff (full architecture) | `PROJECT_HANDOFF.md` (in Jarviscore root) |
| Docs/specs | `/home/billybart/Downloads/Harold/esotericv0.2CURRENT/Harold/docs/specs/` |

---

## Website Overview

The Herald website is a Next.js 15 static-export site (`output: "export"` in next.config.ts).

### Tech Stack
- **Framework**: Next.js 15 (App Router, static export)
- **3D**: Three.js via @react-three/fiber + @react-three/drei
- **Animation**: GSAP + Lenis smooth scroll
- **Styling**: CSS Modules
- **Package manager**: npm

### Site Structure
```
site/New folder/
  src/
    app/
      layout.tsx              # Root layout
      globals.css             # Global styles
      (site)/                 # Main route group
        page.tsx              # Homepage (renders Hero)
        applications/         # Applications page
        business/             # Business case page
        classification/       # Intent classification page
        cognitive-layer/      # Concept D / world model page
        comparison/           # vs ChatGPT/Claude/etc comparison page
        crsis/                # Self-improvement engine page
        how-it-works/         # Architecture walkthrough
        model-seats/          # Model lineup page
        numbers/              # Stats/metrics page
        pattern/              # CLLM pattern page
        problem/              # Problem statement page
        roadmap/              # Future plans page
        skeptic/              # Skeptic FAQ page
        world-model/          # World model deep dive
      explore/                # Interactive explorer
    components/
      Hero.tsx                # Landing hero with 3D scene + typewriter
      Nav.tsx                 # Navigation
      Footer.tsx              # Footer
      HowItWorks.tsx          # Architecture diagram section
      ModelSeats.tsx          # Model lineup cards
      Comparison.tsx          # Competitor comparison table
      CognitiveLayer.tsx      # World model explanation
      Crsis.tsx               # CRSIS self-improvement section
      Classification.tsx      # 5-stage routing visualization
      WorldModel.tsx          # Evidence store / belief state
      Numbers.tsx             # Stats (LOC, models, tools, etc.)
      Roadmap.tsx             # Future plans
      CodeBlock.tsx           # Code snippet display
      GsapSection.tsx         # GSAP scroll animation wrapper
      ScrollReveal.tsx        # Scroll-triggered reveal
      TextReveal.tsx          # Text animation
      PinnedSection.tsx       # Scroll-pinned section
      SmoothScroll.tsx        # Lenis wrapper
      ScrollProgress.tsx      # Scroll progress indicator
      SiteComponents.tsx      # Shared component exports
      Applications.tsx        # Use cases section
      Business.tsx            # Business model section
      Skeptic.tsx             # FAQ / pushback section
      Inversion.tsx           # Design philosophy section
      Problem.tsx             # Problem statement section
      ClosingCta.tsx          # Call to action
  public/
    esoteric.html             # Static Esoteric page
  out/                        # Static export output (pre-built)
  next.config.ts              # Static export config
  package.json                # herald-site, Next.js 15
```

### Build & Run
```bash
cd "/home/billybart/Downloads/Harold/site/New folder"
npm install
npm run dev      # Dev server
npm run build    # Static export to out/
```

---

## What Needs Updating on the Website

The website was built when the model lineup was different and before recent architectural changes. Here is what is now **outdated** and needs to be updated:

### 1. Model Seats (CRITICAL - ModelSeats.tsx)

The `MODEL_SEATS` array in `src/components/ModelSeats.tsx` shows the OLD model lineup. Update to match the current `jarvis/config.py`:

**Current config.py (ground truth):**
```
renderer_model_preferred:  gemma4:e2b        (was llama3.2:1b)
renderer_model_fallback:   gemma4:e4b        (was llama3.2:3b)
vision_lite_model:         gemma4:e2b        (was qwen2.5vl:3b — shared with renderer)
vision_bg1_model:          qwen3-vl:8b       (unchanged)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bibbisalsd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
