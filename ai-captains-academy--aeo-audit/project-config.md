---
trigger: always_on
description: A multi-agent rig that generates AEO (Answer Engine Optimization) and SEO audit reports for businesses. Runs in two modes:
---

# AEO & SEO Audit -- AI Search & SEO Audit Generator Rig

## What This Is

A multi-agent rig that generates AEO (Answer Engine Optimization) and SEO audit reports for businesses. Runs in two modes:

- **Light mode**: Quick research + AEO snapshot + cold outreach email. Costs fewer tokens, produces a cold-outreach asset for initial prospecting. Run this first to see if the prospect bites.
- **Full mode**: Complete 25-35 page branded report, executive summary, cover letter, and follow-up email sequence. Run this when a prospect responds to a light-mode outreach, or when you want the full deliverable package upfront.

A member provides a business URL, competitor URLs (or auto-find), and target location. The rig crawls the site, calls free APIs for real metrics (PageSpeed, TLS/SSL, W3C, WHOIS, Wayback), performs a deep AEO analysis, compares competitors, and produces the selected deliverable package.

**AEO is the lead, not a section.** The AEO assessment is the first thing the prospect sees, the executive summary leads with it, and the cover letter opens with it. This is what makes the audit stand out in a world of commodity SEO audits.

Members use the output to close $2,000-$3,500 deals with the "did the work first" motion: run the audit without asking, send the executive summary cold, close when they ask "can you fix this?"

## Quick Start

- `/setup` -- Configure your branding, niche, pricing, and optional PageSpeed API key (run once)
- `/run-audit` -- Generate an AEO & SEO audit (you'll choose light or full mode)

## The Agent Team

| Agent | Role | When to Use |
|-------|------|-------------|
| **Web Crawler** | Crawls target URL + key pages via WebFetch for HTML analysis, meta tags, schema, headings | Phase 2 of /run-audit |
| **API Caller** | Calls 5 free APIs (PageSpeed, SSL Labs, W3C, Wayback, WHOIS) via Bash curl for real metrics | Phase 2 of /run-audit (parallel with Web Crawler) |
| **Competitor Researcher** | Finds and analyzes 2-3 competitors | Phase 2 of /run-audit (parallel with Web Crawler) |
| **AEO Analyst** | Deep-dive AI search readiness assessment -- the star analysis | Phase 2.5 of /run-audit |
| **SEO Strategist** | Synthesizes all research into scoring framework, keyword gaps, action plan | Phase 3 of /run-audit |
| **Report Writer** | Writes assigned report section (spawned 5x for 5 sections) | Phase 4 of /run-audit |
| **Report Designer** | Creates visual design brief for branded PDF report | Phase 4.5 of /run-audit |
| **Report Builder** | Builds report.html, executive-summary.html, follow-up emails | Phase 5 of /run-audit |
| **Design Reviewer** | Reviews HTML deliverables for layout quality, responsiveness, typography, print rendering | Phase 5.7 of /run-audit |
| **Cover Letter Writer** | Personalized outreach letter leading with AEO findings | Phase 5.5 of /run-audit |
| **Quality Reviewer** | **GATE**: Validates technical accuracy, API data usage, AEO depth, completeness, specificity | Phase 6 of /run-audit (mandatory) |

## Context Engineering (CRITICAL)

### Rules for the Orchestrator (YOU)

1. **NEVER read skill files yourself.** Skills are for agents. They are at `.claude/skills/*/SKILL.md`.
2. **NEVER read agent definition files yourself.** Pass the agent file path in the Task prompt so the agent reads its own instructions. They are at `.claude/agents/*.md`.
3. **Pass file PATHS, not file CONTENTS** to agents. Tell the agent: "Read the file at `<path>`."
4. **Agents write directly to disk.** You do NOT receive generated content back.
5. **Track status, not content.** After an agent finishes, you need: (a) status, (b) file paths, (c) issues. NOT file contents.
6. **Quality Reviewer reads from disk.** Pass it the output directory path.
7. **For report writing, spawn ONE Report Writer per section.** Each spawn writes one file.
8. **Use max_turns on Task calls.** Web Crawler: 18, API Caller: 15, Competitor Researcher: 12, AEO Analyst: 15, SEO Strategist: 15, Report Writers: 20, Report Designer: 15, Report Builder: 30, Design Reviewer: 15, Cover Letter Writer: 10, Quality Reviewer: 25.

### How to Spawn Agents (Context-Safe Pattern)

CORRECT -- agent reads its own files, writes to disk:
```
Task tool:
  subagent_type: "general-purpose"
  prompt: |
    You are the [Agent Name] for the AEO & SEO Audit.
    Read your full instructions at: .claude/agents/<agent>.md

    ## Your Task
    [Brief task description]

    ## Input Files (read these yourself)
    - Plan: output/<client-slug>/plan.md
    - [Other inputs]: <path>

    ## Output Files (write these yourself)
    - <path-to-output>

    ## Return Format
    Return ONLY:
    - Status: SUCCESS or FAILED
    - Files created: [list of paths]
    - Issues: [any problems encountered]
```

WRONG -- orchestrator reads everything and pastes it in:
```
Read .claude/agents/<agent>.md          <- wastes orchestrator context
Read .claude/skills/<skill>/SKILL.md    <- wastes orchestrator context
Task tool with all that content pasted  <- doubled context usage
```

## Workflow: /run-audit

### Phase 1: Input & Planning (MANDATORY)
1. Read `config/member-profile.md` for member branding/niche and optional API key

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Captains-Academy/aeo-audit](https://github.com/AI-Captains-Academy/aeo-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
