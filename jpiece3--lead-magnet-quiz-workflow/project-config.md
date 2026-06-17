---
trigger: always_on
description: Standalone repository for building and deploying quiz funnel and vision board builder lead magnets. This is a packaged version of the lead magnet workflows from Vibe Marketing Studio, designed to run independently with Claude Code.
---

# Lead Magnet Quiz Workflow

Standalone repository for building and deploying quiz funnel and vision board builder lead magnets. This is a packaged version of the lead magnet workflows from Vibe Marketing Studio, designed to run independently with Claude Code.

> **Maintenance Note**: Keep this file updated when modifying skills, agents, or workflow stages. This is the source of truth for the project.

## Project Structure

```
.claude/
  skills/
    lead-magnet-quiz/           # Quiz funnel orchestrator skill
      SKILL.md
      references/               # Builder prompt template, CSV schemas, video templates
    lead-magnet-vision-board/   # Vision board orchestrator skill
      SKILL.md
      references/               # Glif prompt patterns, vertical templates (wedding, real-estate, contractor)
    setup-quiz-db/              # Supabase setup for quiz funnels
      SKILL.md
    setup-visionboard-db/       # Supabase + Glif setup for vision boards
      SKILL.md
agents/
  lead-magnet-agents/           # Quiz funnel agent definitions
    project-manager/            # Stage 0: Validates inputs, checks website access
    builder-agent/
      research-agent/           # Stage 1A: Market research via MCP tools
    quiz-architecture-agent/    # Stage 2A: Question flow, scoring, profiles
    design-strategy-agent/      # Stage 2B: Brand detection, design mode, motion system
      references/               # Component library, decorative elements, motion patterns, shape vocabulary
    copy-agent/                 # Stage 3: Landing page, quiz, emails, strategy pack
    build-agent/                # Stage 4: Astro project, edge functions, social ad video
      references/               # Astro patterns
    shared/                     # Shared utilities (image gen prompts, Playwright utils, question patterns)
    TROUBLESHOOTING.md          # Diagnostics guide
  vision-board-agents/          # Vision board agent definitions
    vb-architecture-agent/      # Preference dimensions, selection flow, profile matching
    vb-copy-agent/              # Builder copy, reveal page, emails, strategy pack
    vb-build-agent/             # Astro project, builder UI, Glif integration
    service-scraping-agent/     # Service/portfolio extraction
shared/
  templates/                    # Output templates (architecture, copy, design, research)
  examples/                     # Reference examples (landing pages, emails, lead magnets, screenshots)
remotion/                       # Remotion video rendering workspace
  src/remotion/                 # Dynamic composition, compiler, webpack config
  src/skills/                   # Video generation skills (3D, charts, typography, etc.)
  render-quiz-videos.sh         # Shell script to render social ad videos
marketing/
  strategy/                     # Sales collateral (ad strategy, call scripts, reel scripts)
  public/                       # Public-facing landing pages
  references/                   # Landing page design samples
output/                         # Working directory for active builds (gitignored)
scripts/
  setup.sh                      # Interactive configuration script
workflow-config.json            # GitHub username, Notion DB ID, paths, pricing
```

## Skills

| Skill | Description |
|-------|-------------|
| `/lead-magnet-quiz` | Orchestrated 6-agent quiz funnel builder with optional video assets |
| `/lead-magnet-vision-board` | Orchestrated 6-agent vision board builder with Glif graphic generation |
| `/setup-quiz-db` | Automate Supabase setup for quiz funnels (run after `/lead-magnet-quiz`) |
| `/setup-visionboard-db` | Automate Supabase + Glif setup for vision boards (run after `/lead-magnet-vision-board`) |

---

## Lead Magnet Quiz Workflow

Multi-agent workflow that produces a Vercel-deployable quiz funnel.

### Stages

- **Stage 1A+1B**: Research Agent + Product Scraping Agent (parallel)
- **Stage 2A+2B**: Architecture Agent + Design Strategy Agent (parallel)
- **Stage 3**: Copy Agent (+ Strategy Pack: ads, social roadmap, sales scripts)
- **Stage 4**: Build Agent (outputs Vercel-ready Astro app with local images + social ad video)
- **Stage 5**: Publish Agent (GitHub + GitHub Pages + Notion)
- **Post-workflow**: Run `/setup-quiz-db [business-name]` to connect Supabase and deploy

### Trigger

```
/lead-magnet-quiz [website-url]
```

### Output Structure

```
[business-name]/
  README.md                    # Overview (root level)
  builder-prompt.md            # AI-ready development prompt (root level)
  deploy/                      # Vercel-ready Astro project
    astro.config.mjs           # Astro config with Vercel adapter
    tsconfig.json, package.json, vercel.json, .env.example
    public/                    # Static assets
      images/                  # Logo + product images (local)
      styles/global.css        # CSS variables from design.md
      scripts/                 # quiz.js, admin.js
    src/                       # Astro source files
      layouts/Layout.astro
      pages/                   # index.astro, quiz/, admin/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jpiece3/lead-magnet-quiz-workflow](https://github.com/jpiece3/lead-magnet-quiz-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
