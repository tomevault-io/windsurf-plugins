---
trigger: always_on
description: Design Guard is a design intelligence CLI and library for AI-generated web design.
---

# Design Guard — Claude Code Development Guide

## Project Overview

Design Guard is a design intelligence CLI and library for AI-generated web design.
It provides business research, design system synthesis, anti-slop validation,
DESIGN.md management, multi-generator support, and integration with Claude Code via skills.

**Stack**: Node.js 20+, TypeScript, Ink (React for terminal), Vitest
**MCP**: `@_davideast/stitch-mcp` for Stitch API access
**Auth**: Stitch API Key (env var `STITCH_API_KEY`) or gcloud OAuth

## Architecture

```
design-guard/                        # Monorepo root (npm workspaces)
├── CLAUDE.md                        # You are here
├── package.json                     # Workspace root — delegates to packages
├── tsconfig.json                    # Project references (core + cli)
├── tsconfig.base.json               # Shared TypeScript compiler options
├── .claude/
│   ├── agents/                      # Claude Code custom agents
│   │   └── dg-critic.md             # Post-generation quality gate agent
│   ├── rules/                       # Anti-slop design rules
│   └── skills/                      # Claude Code skills (dg-* prefix)
├── packages/
│   ├── core/                        # @design-guard/core — design intelligence library
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── vitest.config.ts
│   │   └── src/
│   │       ├── index.ts             # Public API barrel export
│   │       ├── research/
│   │       │   ├── types.ts         # Shared types (BusinessBrief, SiteAnalysis, etc.)
│   │       │   ├��─ business-researcher.ts  # Site scraping + brand analysis
│   │       │   ├── design-synthesizer.ts   # Research -> DESIGN.md synthesis
│   │       │   └── research-cache.ts       # Research result caching
│   │       ├── validation/
│   │       │   ├── design-validator.ts     # DESIGN.md quality scoring
│   │       │   └── output-validator.ts     # HTML output anti-slop validation
│   │       ├── templates/
│   │       │   ├── design-md.ts     # DESIGN.md template generator
│   │       ���   └── prompts.ts       # Prompt templates (initial, refinement, locale)
│   │       └── utils/
│   │           ├── validators.ts    # Zod schemas + prompt validation
│   │           └── prompt-enhancer.ts # Prompt improvement + slop risk scoring
│   └── cli/                         # design-guard CLI — depends on @design-guard/core
│       ├── package.json
│       ├── tsconfig.json
│       ├── vitest.config.ts
│       └── src/
│           ├── index.ts             # CLI entry point (commander + ink)
│           ├── commands/
│           │   ├── init.ts          # dg init — setup project, auth
│           │   ├── design.ts        # dg design — generate/import DESIGN.md
│           │   ├── discover.ts      # dg discover — research + synthesize DESIGN.md
│           │   ├── generate.ts      # dg generate — send prompts to Stitch
│           │   ├── build.ts         # dg build — build Astro site from screens
│           │   ├── preview.ts       # dg preview — local preview server
│           │   ├── research.ts      # dg research — auto-update knowledge base
│           │   ├── sync.ts          # dg sync — pull project state from Stitch
│           │   └── workflow.ts      # dg workflow — multi-step workflows
│           ├── tui/
│           │   ├── App.tsx          # Main Ink app component
│           │   ├── Dashboard.tsx    # Project overview, screen list, quota usage
│           │   ├── PromptBuilder.tsx # Interactive prompt composer with guardrails
│           │   ├── DesignEditor.tsx  # DESIGN.md section editor
│           │   └── components/
│           │       ├── ScreenCard.tsx
│           │       ├── QuotaMeter.tsx
│           │       ├── StatusBar.tsx
│           │       └── Spinner.tsx
│           ├── mcp/
│           │   ├── client.ts        # MCP client wrapper for Stitch tools
│           │   ├── tools.ts         # Tool call builders
│           │   └── auth.ts          # Auth handler (API key or OAuth)
│           ├── adapters/
│           │   ├── index.ts         # Adapter registry
│           │   ├── types.ts         # Adapter types
│           │   ├── astro.ts         # Astro site builder
│           │   ├── static.ts        # Static HTML site builder
│           │   └── nextjs.ts        # Next.js site builder
│           ├── research/
│           │   ├── crawler.ts       # Fetch and parse Stitch docs/changelog
│           │   ├── differ.ts        # Compare current vs new state
│           │   ├── updater.ts       # Apply updates to knowledge base
│           │   └── known-state.json # Last known state of Stitch features
│           ├── templates/
│           │   └── workflows.ts     # Workflow sequences (redesign, new app)
│           └── utils/
│               ├── config.ts        # Project config (.guardrc.json)
│               ├── logger.ts        # Structured logging
│               ├── quota.ts         # Track generation usage against limits
│               └── preview.ts       # Local preview server
├── docs/
│   ├── design-md-guide.md
│   ├── prompting-guide.md
│   └── known-state.md
├── public/                          # Landing page assets
├── scripts/                         # Repo scripts (demo.tape)
├── README.md
├── LICENSE

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FReptar0/design-guard](https://github.com/FReptar0/design-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
