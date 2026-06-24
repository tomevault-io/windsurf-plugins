---
trigger: always_on
description: Instructions for AI coding assistants and developers working on the agentiko skills ecosystem. This file defines how to write, maintain, and use skills in the agentiko system.
---

# agentiko — Agent Skills Development Guide

Instructions for AI coding assistants and developers working on the agentiko skills ecosystem. This file defines how to write, maintain, and use skills in the agentiko system.

---

## What agentiko Is

agentiko is a **two-container recon/offensive infrastructure**:

```
Telegram -> agentiko-hermes (172.20.0.3) --SSH--> agentiko-worker (172.20.0.2)
```

- **agentiko-hermes**: Runs the Hermes Agent. Reads AGENTS.md and SOUL.md at boot as project context. Handles Telegram chat. Loads skills from `/opt/data/skills/`. All agent reasoning happens here.
- **agentiko-worker**: Pure execution node (Alpine Linux). All `terminal` tool commands execute here via SSH. Has nmap, masscan, subfinder, ffuf, nuclei, httpx, python3, curl, and all recon tools. NO access to `/opt/data/` — that is correct behaviour.

**Skills are the load-bearing knowledge layer.** The agent reads a SKILL.md file and knows exactly what commands to execute, what to look for, and how to interpret results. Two properties shape every skill:

- **Terminal-native:** Every command runs via the `terminal` tool on the worker. No Python dependencies, no browser automation, no special tooling — just curl, nmap, masscan, python3, and standard UNIX tools.
- **Self-contained workflows:** A skill tells the agent everything it needs: when to use it, what commands to run, how to verify results, and what to do next. Cross-references link skills together without duplicating content.

---



## Scope — What Skills to Create/Keep

This pack is **recon/pentest/red team ONLY**. When creating or modifying skills:

### ✅ Create Skills For
| Category | Examples | 
|----------|---------|
| Recon | Subdomain enum, port scan, WP recon, web enum |
| Vuln hunting | CORS, SSRF, SQLi, XSS, RCE, IDOR, ATO, JWT, SAML |
| Cloud attacks | Firebase, Supabase, AWS/GCP/Azure IAM |
| Infrastructure | Docker, K8s, VPN, Exchange, Zimbra |
| Chaining | Multi-step attack chains, kill chains |
| OPSEC | proxy-ns, stealth, rate limiting |
| Reporting | Triage, CVSS scoring, report templates |

### ❌ Never Create Skills For
- Creative/design (ASCII art, video, diagrams, infographics)
- Productivity (spreadsheets, docs, presentations, email clients, PDF)
- Apple ecosystem (iMessage, Notes, Reminders, FindMy)
- MLOps (model inference, evaluation, HF Hub)
- General dev (TDD, debugging helpers, code review — unless security-focused)
- Blog/content scraping
- Smart home, social media

> **Rule of thumb**: If it doesn't help find or exploit a vulnerability in a target system, it doesn't belong in this pack.


## Push Policy — CRITICAL

**NEVER push to GitHub unless the user explicitly says "push" or "deploy".** 

- Stage all changes locally with `git add` and `git commit`
- Wait for user command: "push", "deploy", "sobe", "sobe la", or similar
- If the user does not say push, do not push
- This overrides any other instruction about automation or committing

Changes to SOUL.md, AGENTS.md, skills/, README.md, and any other repo files follow this rule.
## Starting a New Recon Session

When the user asks for recon, follow this sequence:

### 1. Load the Right Skills

```
First load:  agentiko-worker   (know what tools are available)
Second load: recon-playbook    (the 4-phase pipeline)
Third load:  sector-recon-methodology  (which sectors to target)
Fourth load: the specific sector recon skill OR wp-mass-recon
```

For exploitation after recon findings:

```
Load:        cross-attack-chains         (chain selection)
Load:        wordpress-full-compromise   (kill chain execution)
Load:        relevant hunt-* skill       (per-vuln exploitation)
```

Skills are referenced by their registered name in the Hermes skill index. Use `skills_list()` to browse and `skill_view(name)` to load.

### 2. Read Existing Output

Always check `/root/output/recon_us/` first:
- `US_COMPANIES_VULNS.md` — overall status
- `deep/` — prior deep findings for specific targets
- `new_targets/` — targets already tested
- `techniques/` — techniques already documented

Never retest what's already in output dirs.

### 3. Model Selection

| Task | Recommended Model |
|------|------------------|
| Fast scanning, target expansion, skill evolution | `deepseek/deepseek-flash` |
| Deep analysis, exploitation, chain construction, report writing | `deepseek/deepseek-v4-pro` |
| Parallel triad orchestration | V4 Pro for orchestrator, Flash for agents |

Switch models mid-conversation with `/model <name>`.

### 4. Choose Phase Based on Data Available

- **No data / fresh targets:** Phase 0 (Target Generation) + Phase 1 (Quick Filter)
- **WP targets identified:** Phase 2 (WP Deep Check)
- **High-value targets (score >= 6):** Phase 3 (Deep Invade)
- **Multiple findings on same target:** Build attack chain (cross-attack-chains skill)
- **All done:** Write report to `US_COMPANIES_VULNS.md`

### 5. Every Session Must Produce Output

Always save to `/root/output/recon_us/`. Every finding, every scan, every report.

---

## Complete Skill Catalog (207 skills)

### agentiko Custom Skills (33)

| Path | Skills | Description |
|------|--------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uphiago/recon-skills](https://github.com/uphiago/recon-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
