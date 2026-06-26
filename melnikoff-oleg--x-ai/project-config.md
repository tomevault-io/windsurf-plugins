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
├── .env                   # API keys (APIFY_API_TOKEN, GEMINI_API_KEY, ANTHROPIC_API_KEY, KIE_AI_API_KEY)
├── .claude/
│   └── commands/          # Slash commands Claude can execute
├── app/                   # X AI — Next.js web application
│   └── src/
│       ├── app/           # Pages + API routes
│       │   ├── layout.tsx         # Root layout (sidebar + top-bar + content)
│       │   ├── globals.css        # Dark theme, glass-morphism, oklch colors
│       │   ├── page.tsx           # Redirects to /posts
│       │   ├── posts/page.tsx     # Post browser with filters, dual-view cards, modals
│       │   ├── creators/page.tsx  # Creator management (X accounts)
│       │   ├── configs/page.tsx   # Pipeline config management
│       │   ├── run/page.tsx       # Pipeline execution with live progress (4 phases)
│       │   └── api/               # API routes (posts, creators, configs, pipeline, proxy-image)
│       ├── lib/           # Core logic
│       │   ├── apify.ts           # X/Twitter scraping via Apify
│       │   ├── gemini.ts          # Media upload + deep multimodal analysis
│       │   ├── claude.ts          # Ready-to-post content + image prompt generation
│       │   ├── kie.ts             # Kie AI infographic image generation
│       │   ├── brand-style.ts     # Brand style constants + reference image helper
│       │   ├── pipeline.ts        # 4-phase orchestrator
│       │   ├── csv.ts             # CSV read/write with typed helpers
│       │   └── types.ts           # All TypeScript interfaces
│       └── components/    # UI components
│           ├── app-sidebar.tsx    # Navigation sidebar
│           ├── top-bar.tsx        # Sticky backdrop-blur header
│           ├── markdown-content.tsx # Markdown renderer
│           └── ui/                # shadcn/ui primitives
├── data/                  # CSV data store (creators.csv, posts.csv, configs.csv)
│   └── generated-images/  # Kie AI generated infographic PNGs
├── references/            # Reference infographic images (5 Jake Ward style examples)
├── scripts/               # One-off utility scripts (generation demos, testing)
├── output/                # Generated output files (HTML comparisons, demos)
├── context/               # Background context about the user and project
└── plans/                 # Implementation plans
```

**Key directories:**

| Directory    | Purpose                                                                             |
| ------------ | ----------------------------------------------------------------------------------- |
| `app/`       | X AI Next.js app — viral content analyzer + generator for marketing/CRO niche. |
| `data/`      | CSV storage for creators, posts, configs, and generated images.                    |
| `references/`| 5 reference infographic images (Jake Ward style) for Kie AI style matching.       |
| `scripts/`   | Generation scripts: `generate-demo.ts` (single post), `generate-batch.ts` (3-post batch with QC), `generate-smart.ts` (10-post with AI-driven style classification). |
| `output/`    | Generated output files (HTML comparison pages, demo artifacts).                   |
| `context/`   | Who the user is, their role, current priorities, strategies. Read by `/prime`.      |
| `plans/`     | Detailed implementation plans. Created by `/create-plan`, executed by `/implement`. |

---

## X AI App

A Next.js web application that analyzes viral X/Twitter posts and generates ready-to-publish content with branded infographic images.

**Run:** `npm run dev` → http://localhost:3000

**Niche:** Marketing, CRO, growth. Pre-configured with "Growth Marketing" config.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melnikoff-oleg/x-ai](https://github.com/melnikoff-oleg/x-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
