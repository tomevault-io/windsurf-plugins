---
trigger: always_on
description: > **Read this file first** at the start of every session. It is the source of truth for Imtihan's architecture, conventions, and open decisions. Update it when anything fundamental changes.
---

# CLAUDE.md

> **Read this file first** at the start of every session. It is the source of truth for Imtihan's architecture, conventions, and open decisions. Update it when anything fundamental changes.

---

## 1. What Imtihan Is

**Imtihan** (امتحان — "exam" in Arabic) is an AI-powered exam generator for teachers in Lebanon, covering school curricula (Bac Libanais, Bac Français, IB) and university courses. Teachers describe what they need, optionally upload course materials, and receive a polished exam + corrigé in Word and PDF formats.

**Owner:** Founder — solo founder, based in Bsabba, Lebanon.
**Status:** MVP in development. Not yet launched.
**Target launch:** Q3 2026.

## 2. The Core Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  STEP 1 — DESCRIBE                                              │
│  Teacher writes a natural-language description + optionally     │
│  uploads PDFs/DOCX/images (textbook chapter, past exam, notes). │
│                                                                 │
│  → POST /api/analyze                                            │
│  → Gemini 2.5 Flash with vision reads everything                │
│  → Returns structured ExamContext (curriculum, level, subject,   │
│    chapters, language, duration, exercise count, etc.)          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 2 — CONFIRM CONTEXT                                       │
│  Auto-filled form. Teacher reviews/edits each field.            │
│  Validates against curriculum JSON (chapter must exist in level)│
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 3 — STRUCTURE & STYLE                                     │
│  Points distribution, difficulty mix slider, template choice,   │
│  Version A/B toggle.                                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 4 — GENERATE & REFINE                                     │
│  POST /api/generate — streams exercises from Claude.            │
│  Per-exercise actions: regenerate, edit, easier, harder, remove.│
│  Corrigé generated alongside, toggleable view.                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  STEP 5 — EXPORT                                                │
│  Download Word (docx lib), download PDF (@react-pdf/renderer),  │
│  or email. Auto-saved to Firestore under users/{uid}/exams.     │
└─────────────────────────────────────────────────────────────────┘
```

## 3. Tech Stack (Locked In)

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 15 (App Router) + React 19 | SEO, server components, API routes in same repo |
| Language | TypeScript (strict) | End-to-end type safety |
| Styling | Tailwind CSS v3.4 | Fast iteration, design tokens in `tailwind.config.ts` |
| Components | Custom — no shadcn | Editorial aesthetic, no generic look |
| Fonts | Fraunces (display serif) + Geist (sans) | Distinctive, not the usual Inter |
| Animation | Framer Motion | For stage transitions and reveals |
| Auth | Firebase Auth | Standard cloud stack |
| Database | Firestore | Teacher accounts, saved exams, exam library |
| Storage | Firebase Storage | Uploaded source documents, generated files |
| AI | Google Gemini 1.5 Flash | Best quality/cost for exam generation + vision |
| Word export | `docx` npm package | Full styling control |
| PDF export | `@react-pdf/renderer` | React-native PDF rendering |
| Payments | Stripe (v1.1+) | Monthly subscription, USD |
| Hosting | Vercel | Next.js native |

**Do not swap these without updating this file and getting approval.**

## 4. Directory Conventions

```
src/
├── app/                  # Next.js App Router — routes + API endpoints
│   ├── (marketing)/      # Public pages: landing, pricing, about
│   ├── (app)/            # Authenticated app: dashboard, create, library
│   └── api/              # Route handlers
├── components/
│   ├── ui/               # Primitives: Button, Input, Dropzone
│   ├── landing/          # Marketing-only sections
│   ├── workflow/         # The 5-step exam creation flow
│   └── layout/           # Header, Footer, Shell
├── lib/
│   ├── anthropic.ts      # Claude API client
│   ├── firebase.ts       # Client SDK init
│   ├── firebase-admin.ts # Server SDK init
│   ├── utils.ts          # cn(), formatters
│   └── prompts/          # System prompts — versioned, testable
├── types/                # Shared TS types: Exam, ExamContext, etc.
└── data/
    └── curricula/        # Curriculum JSON — single source of truth
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AntoineKhawand/imtihan](https://github.com/AntoineKhawand/imtihan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
