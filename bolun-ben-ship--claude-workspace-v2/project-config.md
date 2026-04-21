---
trigger: always_on
description: The central workspace for RightClick:AI's SEO operations.
---

# RightClick:AI — Agency Workspace

## What This Is
The central workspace for RightClick:AI's SEO operations.
Contains the skill library, client registry, onboarding template, and agency-level outputs.

This is NOT a client workspace — open client subfolders in `clients/` separately as their own Claude Code project.

---

## Workspace Structure

```
RightClickAI-seo-workspace/
├── CLAUDE.md                          ← this file
├── clients.md                         ← client registry (all clients, platforms, status)
├── clients/                           ← one subfolder per client
│   └── {domain}/                      ← open THIS in Claude Code when working on a client
│       ├── CLAUDE.md                  ← client config (platform, handle, token env var)
│       ├── {client-slug}-*.json       ← Google service account key (per-client, gitignored)
│       ├── .claude/
│       │   └── commands/
│       │       ├── start-client.md    ← /start-client command (reference copy per client)
│       │       └── prime.md           ← /prime command (reference copy per client)
│       ├── context/
│       │   ├── client-info.md         ← brand, products, competitors, audience
│       │   ├── tone-guide.md          ← brand voice, writing rules (read by blog-write)
│       │   ├── current-data.md        ← live SEO coverage stats, analytics baseline
│       │   ├── strategy.md            ← content & growth strategy
│       │   └── personal-info.md       ← founder voice, values (optional)
│       ├── scripts/                   ← client automation scripts
│       │   └── mailchimp-sync.sh      ← Webflow + Calendly → Mailchimp lead sync (hourly)
│       ├── Content & SEO/
│       │   └── outputs/
│       │       └── {platform}-{handle}/
│       │           ├── audit/         ← AUDIT-YYYY-MM-DD.md
│       │           ├── implementation/← SEO-PLAN-YYYY-MM-DD.md, SNAPSHOT-YYYY-MM-DD.md
│       │           ├── research/      ← GSC-REPORT, GA4-REPORT, SOCIAL-TRENDS
│       │           ├── keywords/      ← KEYWORDS-YYYY-MM-DD.md
│       │           ├── blog-plans/    ← BLOG-PLAN-YYYY-MM-DD.md
│       │           └── blogs/         ← blog HTML posts
│       └── Design/
│           └── Carousel-YYYY-MM-DD/   ← /carousel output (HTML preview + PNG slides)
├── client-template/                   ← copy of this scaffolded by /onboard
│   ├── CLAUDE.md
│   ├── .claude/
│   │   └── commands/
│   │       ├── start-client.md        ← /start-client source (deployed globally by install.sh)
│   │       └── prime.md               ← /prime source (deployed globally by install.sh)
│   ├── context/
│   │   ├── client-info.md
│   │   ├── current-data.md
│   │   └── strategy.md
│   └── Content & SEO/outputs/
├── context/                           ← agency-level context
│   ├── agency-info.md                 ← what RightClick:AI does, skill stack
│   └── agency-strategy.md            ← agency goals and priorities
├── SKILLS-REFERENCE.md                ← master reference for all skills, agents, commands
├── outputs/                           ← agency-level implementation plans and reports
│   └── IMPLEMENTATION-PLAN-YYYY-MM-DD.md
├── seo-workflow/                      ← skill library (source of truth)
│   ├── install.sh                     ← deploys skills → ~/.claude/skills/, agents → ~/.claude/agents/, commands → ~/.claude/commands/
│   ├── agents/                        ← subagent configs
│   ├── carousel/SKILL.md             ← Instagram carousel generator
│   └── {skill-name}/SKILL.md         ← one folder per skill
└── .claude/
    └── commands/
        ├── start-agency.md            ← /start-agency command
        ├── onboard.md                 ← /onboard command
        └── initialise.md              ← /initialise command
```

---

## Skill Library — Source of Truth

`seo-workflow/` is the single source of truth for all skills:
- **Local use** → run `seo-workflow/install.sh` to deploy to `~/.claude/skills/`
- **GitHub** → commit and push `seo-workflow/` to version-control all skills
- **New machine** → pull repo → run `install.sh` → all skills ready

`install.sh` deploys three things:
1. **Skills** → `~/.claude/skills/` (24 skills)
2. **Agents** → `~/.claude/agents/` (6 subagents)
3. **Global commands** → `~/.claude/commands/` (`start-client.md`, `prime.md`)

Global commands are sourced from `client-template/.claude/commands/` and deployed to `~/.claude/commands/` so they appear in ALL Claude Code projects — including client subfolders that share this git root.

### Rules
- **Always edit skills in `seo-workflow/`** — never directly in `~/.claude/skills/`
- **Always edit commands in `client-template/.claude/commands/`** — never directly in `~/.claude/commands/`
- `~/.claude/skills/`, `~/.claude/agents/`, `~/.claude/commands/` are deployment targets — overwritten on every `install.sh` run
- After any skill or command edit, run `bash seo-workflow/install.sh` to deploy

---

## Machine Setup — Global Permissions Required for Automation

The `/ai-seo-pipeline` skill creates Claude Code scheduled tasks that run unattended.
These tasks will pause and prompt for tool approval on every run unless the following
permissions are set globally in `~/.claude/settings.json`:

```json
{
  "permissions": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bolun-ben-ship) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
