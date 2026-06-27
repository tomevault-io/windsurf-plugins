---
trigger: always_on
description: Use when adding new functionality, commands, scripts, or making structural changes. Produces a thorough plan document in `plans/` that captures context, rationale, and step-by-step tasks.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Is

This is a **Claude Workspace Template** — a structured environment designed for working with Claude Code as a powerful agent assistant across sessions. The user will spin up fresh Claude Code sessions repeatedly, using `/prime` at the start of each to load essential context without bloat.

**This file (CLAUDE.md) is the foundation.** It is automatically loaded at the start of every session. Keep it current — it is the single source of truth for how Claude should understand and operate within this workspace.

---

## The Claude-User Relationship

Claude operates as an **agent assistant** with access to the workspace folders, context files, commands, and outputs. The relationship is:

- **User**: Defines goals, provides context about their role/function, and directs work through commands
- **Claude**: Reads context, understands the user's objectives, executes commands, produces outputs, and maintains workspace consistency

Claude should always orient itself through `/prime` at session start, then act with full awareness of who the user is, what they're trying to achieve, and how this workspace supports that.

---

## Workspace Structure

```
.
├── CLAUDE.md              # This file — core context, always loaded
├── package.json           # Root scripts (npm run dev proxies into app/)
├── .env                   # API keys (APIFY_API_TOKEN, GEMINI_API_KEY, ANTHROPIC_API_KEY)
├── .claude/
│   └── commands/          # Slash commands Claude can execute
├── app/                   # TikTok AI — Next.js web application
│   └── src/
│       ├── app/           # Pages + API routes
│       │   ├── layout.tsx         # Root layout (sidebar + top-bar + content)
│       │   ├── globals.css        # Dark theme, glass-morphism, oklch colors
│       │   ├── page.tsx           # Redirects to /videos
│       │   ├── videos/page.tsx    # Video browser with filters, modals
│       │   ├── creators/page.tsx  # Creator management
│       │   ├── configs/page.tsx   # Pipeline config management
│       │   ├── run/page.tsx       # Pipeline execution with live progress
│       │   └── api/               # API routes (videos, creators, configs, pipeline, proxy-image)
│       ├── lib/           # Core logic
│       │   ├── apify.ts           # TikTok scraping via Apify
│       │   ├── gemini.ts          # Video upload + multimodal analysis
│       │   ├── claude.ts          # Concept generation with Claude
│       │   ├── pipeline.ts        # 3-phase orchestrator
│       │   ├── csv.ts             # CSV read/write with typed helpers
│       │   └── types.ts           # All TypeScript interfaces
│       └── components/    # UI components
│           ├── app-sidebar.tsx    # Navigation sidebar
│           ├── top-bar.tsx        # Sticky backdrop-blur header
│           ├── markdown-content.tsx # Markdown renderer
│           └── ui/                # shadcn/ui primitives
├── data/                  # CSV data store (creators.csv, videos.csv, configs.csv)
├── context/               # Background context about the user and project
└── plans/                 # Implementation plans
```

**Key directories:**

| Directory    | Purpose                                                                             |
| ------------ | ----------------------------------------------------------------------------------- |
| `app/`       | TikTok AI Next.js app — viral content analyzer for beauty/lifestyle/fashion niche. |
| `data/`      | CSV storage for creators, videos, and configs. Shared between app and workspace.    |
| `context/`   | Who the user is, their role, current priorities, strategies. Read by `/prime`.      |
| `plans/`     | Detailed implementation plans. Created by `/create-plan`, executed by `/implement`. |

---

## TikTok AI App

A Next.js web application that analyzes viral TikTok content and generates content concepts. Port of the Instagram Social Media AI app (github.com/melnikoff-oleg/social-media), matching its exact dark glass-morphism aesthetic.

**Run:** `npm run dev` → http://localhost:3000

**Niche:** Beauty, lifestyle, fashion. Pre-configured with "Bella Glow" brand persona.

**Pipeline:** Apify (scrape TikTok) → Gemini (analyze video) → Claude (generate concepts)

### Pages

- `/videos` — Browse analyzed videos with glass cards, 9:16 thumbnails, star toggle, Analysis/Concepts modal with underline tabs, full markdown rendering
- `/creators` — Manage TikTok creators with profile pics, stat boxes (followers, hearts, videos), hover-reveal actions
- `/configs` — Manage analysis/concept prompts with preview boxes, creator/video counts per config
- `/run` — Execute pipeline with collapsible advanced settings, gradient progress bar, collapsible log, completion CTA

### Tech Stack

- **Next.js 16.2.4** with Turbopack, App Router
- **TypeScript**, **Tailwind CSS v4**
- **shadcn/ui** with base-ui primitives (NOT Radix — Next.js 16 breaking change)
- **CSV file-based storage** in `data/` directory
- **SSE** (Server-Sent Events) for pipeline progress streaming


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melnikoff-oleg/tiktok-ai](https://github.com/melnikoff-oleg/tiktok-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
