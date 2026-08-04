---
trigger: always_on
description: **Purpose:** Single source of truth for all AI coding assistants (Claude Code, Cursor AI, OpenAI Codex, Google Gemini, GitHub Copilot, and others). Ensures all agents work with consistent guidelines and patterns.
---

# AGENTS.md - Documentation for AI Agents

**Purpose:** Single source of truth for all AI coding assistants (Claude Code, Cursor AI, OpenAI Codex, Google Gemini, GitHub Copilot, and others). Ensures all agents work with consistent guidelines and patterns.

## Detailed Documentation

**Comprehensive guides for specific tasks:**

| Category | Guide | Purpose |
|----------|-------|---------|
| Architecture | [Architecture](docs/ARCHITECTURE.md) | Components, Content Collections, Svelte integration, project structure |
| Standards | [Standards](docs/STANDARDS.md) | Canonical coding rules, orthography, import order |
| Testing | [Testing](docs/TESTING_GUIDE.md) | Vitest setup, conventions, writing tests |
| Commands | [Development Commands](docs/DEVELOPMENT_COMMANDS.md) | npm scripts, Astro CLI, build workflows |
| i18n | [I18N Guide](docs/I18N_GUIDE.md) | Adding languages, translation workflow |
| Performance | [Performance](docs/PERFORMANCE.md) | Astro SSG optimization, image handling, caching |
| Accessibility | [Accessibility](docs/ACCESSIBILITY.md) | WCAG AA, contrast ratios, ARIA patterns |
| SEO | [SEO](docs/SEO.md) | Meta tags, structured data, multilingual SEO, AEO |
| Security | [Security](docs/SECURITY.md) | Static site security best practices |
| Documentation | [Documentation Guide](docs/DOCUMENTATION_GUIDE.md) | When and how to update docs |
| Product | [Product Spec](docs/PRODUCT_SPEC.md) | Product vision, features, website goals |
| Brand | [Brand Guide](docs/BRAND_GUIDE.md) | "Broadsheet" editorial identity: warm paper, ink serif, oxblood accent, editorial primitives |
| Design System | [DESIGN.md](docs/DESIGN.md) | Agent-facing design-system spec (DWP `design-system` addon): Broadsheet tokens (color/type/spacing/radius), component patterns, WCAG do's & don'ts. **Read before generating or editing UI.** |
| Analytics | [Analytics](docs/ANALYTICS.md) | Tracking, GSC, verification |
| AI Agents | [Agent Onboarding](docs/AI_AGENT_ONBOARDING.md), [Agent Collaboration](docs/AI_AGENT_COLLAB.md) | Setup, handoff, coordination |
| Skills/Agents | [Skills & Agents Catalog](.agents/docs/skills_agents_catalog.md) | Available skills and agents |
| Commands | [Commands Reference](.agents/docs/COMMANDS_REFERENCE.md) | All slash commands with procedure files |

## Project Overview

**Deep Work Plan** ([deepworkplan.com](https://deepworkplan.com)) — The official website for the Deep Work Plan (DWP) methodology: the methodology that turns any repository into an **AI-first, agent-pilotable** codebase. DWP documents a repo (AGENTS.md, docs, `.agents/` skills, the DWP skill), enables **long-horizon plans**, and lets **any agent pilot the repo** against explicit acceptance criteria and validation gates. A serious, neutral, fast documentation-and-marketing site built with Astro, in the **"Broadsheet" editorial design system** (warm paper, ink serif display, hairline rules, restrained oxblood accent), with dark mode, multilingual content in 17 languages (en, es, pt, zh, ja, de, fr, ko, ru, it, tr, id, vi, hi, pl, uk, th), static site architecture deployed to Cloudflare Pages.

The site explains and positions the DWP methodology, hosts the readable specification and kit catalog, ships agent-friendly Markdown endpoints, and exposes a canonical adoption endpoint at **`/init`**. The repository **dogfoods** the methodology it documents.

**Positioning — two narrative pillars** (weave through all copy):

1. **Spec-driven development (SDD).** The plan/spec is the durable source of truth; agents execute against explicit acceptance criteria and validation gates (reduces drift, enables verification, survives across sessions/agents). DWP's plan → atomic tasks → gates → completion loop *is* SDD — and unlike tool-bound SDD (GitHub Spec Kit, Amazon Kiro, Tessl), DWP is **tool-agnostic and repo-native**.
2. **Harness engineering — the repository is the harness.** DWP delivers the agent scaffolding (context, tools, control loop, guardrails, state/resumability) as a **portable methodology + kit installed into the repository**, not a per-tool framework — so any agent can pilot any repo. One-liner: *"Deep Work Plan is spec-driven development where the repository itself becomes the harness."*

**Companion skill repo:** the site is paired with **[`DailybotHQ/deepworkplan-skill`](https://github.com/DailybotHQ/deepworkplan-skill)** (DWP packaged as an installable agent skill). Adoption messaging (the `/init` endpoint, methodology, and kit) stays in sync with that repo: install the skill → onboard the agent → generate and execute long-horizon plans. Do **not** attribute the DWP name to any external author or popular-productivity source; the DWP name stands on its own (focused, long-horizon agent execution). Design-system reference: **[Brand Guide](docs/BRAND_GUIDE.md)** (palette + serif type + editorial primitives in `src/components/editorial/`).

**Content model:** methodology documentation is primary, paired with the specification reader and kit catalog. The blog engine, slides/tech-talks, and personal pages have been removed — this is a focused methodology-and-marketing site.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DailybotHQ/deepworkplan-website](https://github.com/DailybotHQ/deepworkplan-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
