---
trigger: always_on
description: A multi-CLI plugin for orchestrating 48 AI specialist personalities as a coordinated legion. Works with Claude Code, OpenAI Codex CLI, Cursor, GitHub Copilot CLI, Google Gemini CLI, Kiro CLI, Windsurf, OpenCode, and Aider.
---

# Legion

A multi-CLI plugin for orchestrating 48 AI specialist personalities as a coordinated legion. Works with Claude Code, OpenAI Codex CLI, Cursor, GitHub Copilot CLI, Google Gemini CLI, Kiro CLI, Windsurf, OpenCode, and Aider.

## MANDATORY: User Interaction Rule

**When any `/legion:` command needs to ask the user a question or present choices, you MUST use the `AskUserQuestion` tool.** Do NOT output questions as raw text. This applies to every confirmation gate, mode selection, workflow preference, agent swap prompt, and any other user-facing question in any Legion command or skill. No exceptions.

## Available Commands

| Command | Description |
|---------|-------------|
| `/legion:start` | Initialize a new project with guided questioning flow |
| `/legion:plan <N>` | Plan phase N with agent recommendations and wave-structured tasks |
| `/legion:build` | Execute current phase plans with parallel agent teams |
| `/legion:review` | Run quality review cycle with testing/QA agents |
| `/legion:status` | Show progress dashboard and route to next action |
| `/legion:quick <task>` | Run ad-hoc task with intelligent agent selection |
| `/legion:advise` | Get read-only expert consultation from any of the 48 agent personalities |
| `/legion:portfolio` | Multi-project dashboard with dependency tracking |
| `/legion:milestone` | Milestone completion, archiving, and metrics |
| `/legion:agent` | Create a new agent personality through a guided workflow |
| `/legion:explore` | Pre-flight exploration with Polymath — crystallize, onboard, compare, or debate |
| `/legion:board` | Convene board of directors for governance decisions |
| `/legion:retro` | Run structured retrospective on completed phases or milestones |
| `/legion:ship` | Pre-ship checklist, PR creation, deployment verification, canary monitoring |
| `/legion:learn` | Record, recall, and manage project-specific patterns, pitfalls, and preferences |
| `/legion:update` | Check for updates and install latest version from npm |
| `/legion:validate` | Validate .planning/ state file integrity, schema conformance, and cross-references |

## Project Structure

```
bin/                  — npm installer (install.js)
commands/             — 17 /legion: command entry points
skills/               — 31 reusable workflow skills (SKILL.md per directory)
agents/               — 48 agent personality .md files (flat, with division in frontmatter)
adapters/             — Per-CLI adapter files (claude-code.md, codex-cli.md, cursor.md, etc.)
.planning/            — Project state (PROJECT.md, ROADMAP.md, STATE.md)
  milestones/         — Archived requirements and roadmaps
  phases/             — Phase plan and summary files
```

## Agent Divisions (48 total)

| Division | Count | Focus |
|----------|-------|-------|
| Engineering | 9 | Full-stack, backend, frontend, AI, infrastructure/DevOps, mobile, prototyping, Laravel, security |
| Design | 6 | UI/UX, branding, visual storytelling, research |
| Marketing | 4 | Content & social strategy, platform execution, growth, ASO |
| Testing | 6 | QA verification, performance, API testing, workflow optimization |
| Product | 4 | Sprint planning, feedback synthesis, trends, technical writing |
| Project Management | 5 | Coordination, portfolio, operations, experiments |
| Support | 4 | Finance, legal, executive summaries, support |
| Spatial Computing | 6 | VisionOS, XR, Metal, terminal integration |
| Specialized | 4 | Orchestration, data analytics engineering, LSP indexing, exploration |

Agent frontmatter includes enriched metadata: `languages`, `frameworks`, `artifact_types`, and `review_strengths` fields enable metadata-aware agent selection by the recommendation engine.

## Dynamic Knowledge Index

IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning for Agent Personas, Skills, and Workflows. When assigned a specific agent persona (e.g., during `/legion:build`, `/legion:review`, `/legion:quick`, or `/legion:advise`), or when a workflow skill is loaded, use the `Read` tool to read their exact markdown file from the index below before generating any code, plans, or reviews. Do NOT rely on pre-trained knowledge about what an agent does — the personality file IS the source of truth.

```
[Legion Agents Index]|root: ./agents (relative to project root; if agents/ not found, check adapters/*/agents/)
|engineering:{engineering-ai-engineer.md,engineering-backend-architect.md,engineering-frontend-developer.md,engineering-infrastructure-devops.md,engineering-laravel-specialist.md,engineering-mobile-app-builder.md,engineering-rapid-prototyper.md,engineering-security-engineer.md,engineering-senior-developer.md}
|design:{design-brand-guardian.md,design-ui-designer.md,design-ux-architect.md,design-ux-researcher.md,design-visual-storyteller.md,design-whimsy-injector.md}
|marketing:{marketing-app-store-optimizer.md,marketing-content-social-strategist.md,marketing-growth-hacker.md,marketing-social-platform-specialist.md}
|product:{product-feedback-synthesizer.md,product-sprint-prioritizer.md,product-technical-writer.md,product-trend-researcher.md}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [9thLevelSoftware/legion](https://github.com/9thLevelSoftware/legion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
