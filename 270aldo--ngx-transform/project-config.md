---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Versión:** 11.0 (Genesis Doctrine)
**Stack:** Next.js 16.0.7 + React 19 + TypeScript + Firebase + Tailwind CSS v4 + Upstash Redis

NGX Transform is a **premium viral lead magnet** that creates realistic 12-month physical transformation projections. Users upload a photo, provide profile data, and receive AI-generated insights with visualized progress images at m0/m4/m8/m12 milestones, plus a personalized 7-day fitness plan.

**v11.0 Genesis Doctrine**: GENESIS is the **sole AI entity visible to the user**. Internally, 13 specialized modules power 4 capabilities (Entrenamiento, Nutricion, Recuperacion, Habitos). Users never see module names — only "GENESIS" and its capabilities.

## Business Context

**Strategic Purpose**: NGX Transform is a **viral lead-generation tool** designed to capture users and convert them to NGX's main subscription fitness app.

**Growth Strategy**:
- Free tool with high shareability (transformation results are inherently viral)
- Captures email leads at wizard entry
- Results page includes CTA to NGX subscription app
- Shareable URLs with Open Graph meta tags for social spread

**Differentiators**:
- Not just "before/after" - temporal projection with narrative (m0→m4→m8→m12)
- Mental + Physical analysis (stress, sleep, discipline factors)
- Cinematic "Nike commercial" aesthetic vs generic fitness apps
- Uses YOUR actual photo, not generic avatars

## User Journey

```
Landing (/)
    ↓
Wizard (/wizard)
    ├── Email capture (lead)
    ├── Photo upload (dropzone)
    ├── Identity & Biometrics (age, sex, height, weight, bodyType)
    ├── Focus Zone (upper/lower/abs/full)
    ├── Goals & Strategy (level, goal, weeklyTime)
    └── Mental Logs (stress, sleep, discipline sliders)
    ↓
Processing (BiometricLoader)
    ├── "Iniciando escaneo biométrico..."
    ├── "Analizando densidad muscular..."
    ├── "Proyectando estructura ósea..."
    └── Motivational tips rotation
    ↓
Results (/s/[shareId])
    ├── CinematicViewer (fullscreen immersive)
    ├── Timeline navigation (HOY → MES 4 → MES 8 → MES 12)
    ├── TransformationSummary (stats delta)
    ├── NeonRadar stats visualization
    └── CTA: "Ver cómo GENESIS crea tu plan"
    ↓
Genesis Demo (/s/[shareId]/demo) — v11.0
    ├── AgentOrchestration (GENESIS central + 4 capability cards)
    │   ├── Phase 1: Analyzing profile (internal: GENESIS, STELLA, LOGOS)
    │   ├── Phase 2: Entrenamiento + Nutricion progress (internal: BLAZE, TEMPO, ATLAS, SAGE, MACRO, METABOL)
    │   └── Phase 3: Recuperacion + Habitos progress (internal: WAVE, SPARK, NOVA, LUNA)
    └── DemoChat (5 interactions, all from GENESIS with capability labels)
    ↓
Plan Preview (/s/[shareId]/plan)
    ├── Day 1: Complete and functional (WorkoutCard, MealPlan, Checklist)
    ├── Days 2-7: Blurred + locked
    └── ComparisonCTA ("Sin GENESIS vs Con GENESIS")
    ↓
Conversion
    └── CTA: "DESBLOQUEAR MI PLAN COMPLETO"
```

## Development Commands

All commands run from `app/`:

```bash
cd app
pnpm dev          # Dev server at localhost:3000
pnpm build        # Production build
pnpm start        # Production server
pnpm lint         # ESLint
```

## Architecture

**Stack**: Next.js 16.0.7 (App Router) + React 19 + Firebase + Google Gemini + Tailwind CSS v4

### Data Flow

1. **Wizard** (`/wizard`) → User uploads photo + profile data
2. **Session Creation** → Firebase Storage (photo) + Firestore (session doc)
3. **Analysis** (`/api/analyze`) → Gemini generates insights with 4-stage timeline
4. **Image Generation** (`/api/generate-images`) → Gemini Image API creates m4/m8/m12 transformations
5. **Results** (`/s/[shareId]`) → Shareable results page with timeline viewer

### Key Services

| Service | Location | Purpose |
|---------|----------|---------|
| `gemini.ts` | `src/lib/` | Gemini 2.5 Flash for profile analysis + user_visual_anchor + style_profile |
| `nanobanana.ts` | `src/lib/` | Gemini Image API with Identity Chain for consistent transformations |
| `firebaseAdmin.ts` | `src/lib/` | Server-side Firestore/Storage operations |
| `storage.ts` | `src/lib/` | Signed URL generation, buffer uploads |
| `validators.ts` | `src/lib/` | Zod schemas for all API inputs |
| `telemetry.ts` | `src/lib/` | Funnel event tracking (wizard_start → cta_completed) |
| `jobManager.ts` | `src/lib/` | Idempotent, resumable jobs for image generation |
| `imageConfig.ts` | `src/lib/` | Centralized image config (model, aspect ratio, size) |
| `promptBuilder.ts` | `src/lib/` | Robust prompt constructor with Identity Lock |
| `qualityGates.ts` | `src/lib/` | Output validation (face visible, single subject, no artifacts) |
| `viral/*` | `src/lib/viral/` | Share-to-unlock, referral tracking, social pack generator |
| `plan/*` | `src/lib/plan/` | 7-day plan generation with AI + templates |
| `schemas/*` | `src/lib/schemas/` | Strict Zod schemas for analysis output |
| `firebaseClient.ts` | `src/lib/` | Client-side Firebase SDK (auth + storage) |
| `emailScheduler.ts` | `src/lib/` | Email nurture sequence scheduling (D0-D7) |
| `watermark.ts` | `src/lib/` | Image watermarking with Sharp |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [270aldo/ngx-transform](https://github.com/270aldo/ngx-transform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
