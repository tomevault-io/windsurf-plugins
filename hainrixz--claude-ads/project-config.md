---
trigger: always_on
description: This repository contains **Claude Ads**, a Tier 4 Claude Code skill focused on the
---

# Claude Ads: Free-first Paid Advertising Audit & Optimization Skill

## Project Overview

This repository contains **Claude Ads**, a Tier 4 Claude Code skill focused on the
3 base advertising platforms where 95% of ad spend lives: **Meta, Google, TikTok**.
It follows the Agent Skills open standard and the 3-layer architecture (directive,
orchestration, execution). After the v2.4.0 guided-onboarding refactor, the skill has
19 sub-skills, 7 agents (3 audit + 4 creative), 8 industry templates, ~158 weighted
audit checks + 3 cross-platform checks, **plus a guided first-run wizard (`/ads start`)
and a continuous coach (`/ads next`) backed by a persistent user profile + audit
history at `~/.claude-ads/`**.

## Architecture

```
claude-ads/
  CLAUDE.md                          # Project instructions (this file)
  ads/                               # Main orchestrator skill
    SKILL.md                         # Entry point, routing table, core rules
    references/                      # On-demand knowledge files
  skills/                            # 19 specialized sub-skills (v2.4.0)
    ads-start/SKILL.md            # NEW v2.4.0 — guided first-run wizard
    ads-next/SKILL.md             # NEW v2.4.0 — continuous coach
    ads-audit/SKILL.md              # Full multi-platform audit
    ads-google/SKILL.md            # Google Ads deep analysis (includes YouTube)
    ads-meta/SKILL.md              # Meta/Facebook Ads analysis — MCP-wired
    ads-tiktok/SKILL.md           # TikTok Ads analysis
    ads-creative/SKILL.md         # Cross-platform creative quality
    ads-landing/SKILL.md          # Landing page analysis
    ads-budget/SKILL.md           # Budget allocation optimization
    ads-plan/SKILL.md             # Strategic ad planning by industry
    ads-competitor/SKILL.md       # Competitor ad research
    ads-dna/SKILL.md              # Brand DNA extraction
    ads-create/SKILL.md           # Campaign briefs / copy decks
    ads-generate/SKILL.md         # AI image generation
    ads-photoshoot/SKILL.md       # Product photography (5 styles)
    ads-math/SKILL.md             # PPC calculator
    ads-test/SKILL.md             # A/B test design
    ads-update/SKILL.md           # Self-refresh platform references
    ads-publish/SKILL.md          # Zernio publishing (first 2 accounts free)
  agents/                            # 7 agents (3 audit + 4 creative)
    audit-google.md                # Google Ads audit (80 checks, includes YouTube)
    audit-meta.md                  # Meta Ads audit (50 checks, MCP-wired)
    audit-tiktok.md                # TikTok Ads audit (28 checks, NEW in v2.3.0)
    creative-strategist.md         # Campaign concept agent
    visual-designer.md             # Image generation agent
    copy-writer.md                 # Copy generation agent
    format-adapter.md              # Asset dimension validation
  scripts/api/                       # Capa-2 direct-API adapters (NEW in v2.3.0)
    meta_fetch.py · google_fetch.py · tiktok_fetch.py · README.md
  scripts/profile.py                 # NEW v2.4.0 — profile + audit history CLI (stdlib)
  install.sh / install.ps1          # Cross-platform installers
  uninstall.sh / uninstall.ps1      # Cross-platform uninstallers

# User-local state (NEW in v2.4.0, never in the repo):
  ~/.claude-ads/profile.json         # validates against ads/references/profile-schema.json
  ~/.claude-ads/history/index.json   # validates against ads/references/audit-history-schema.json
  ~/.claude-ads/history/<platform>-<YYYYMMDDhhmmss>.json
```

## Commands

| Command | Purpose |
|---------|---------|
| `/ads start` | **First-run wizard** (NEW v2.4.0) — context capture + step-by-step OAuth/MCP setup + profile persistence |
| `/ads next` | **Continuous coach** (NEW v2.4.0) — top 3 Quick Wins from audits + regression detection |
| `/ads audit` | Full multi-platform audit with 3 parallel agents (Meta, Google, TikTok) |
| `/ads google` | Google Ads deep analysis (includes YouTube video campaigns) |
| `/ads meta` | Meta/Facebook Ads analysis (MCP-wired to claude.ai Facebook) |
| `/ads tiktok` | TikTok Ads analysis (Smart+, Shop, Symphony, GMV Max) |
| `/ads creative` | Creative quality and fatigue assessment |
| `/ads landing` | Landing page conversion analysis |
| `/ads budget` | Budget allocation optimization |
| `/ads plan <type>` | Strategic ad planning by industry |
| `/ads competitor` | Competitor ad research |
| `/ads math` | PPC financial calculator (CPA, ROAS, break-even, LTV:CAC) |
| `/ads test` | A/B test design (hypothesis, significance, sample size) |
| `/ads report` | PDF audit report generation for client deliverables |
| `/ads update <plat\|all>` | Refresh per-platform references with last 30 days of changes |
| `/ads publish` | Publish generated creatives to 14+ social networks via Zernio (first 2 accounts free) |

## Removed in v2.3.0

The fork upstream covered 7 platforms; v2.3.0 refocuses on the 3 base. **Removed:**

- `ads-apple/` — no MCP exists for Apple Ads; partner-API setup high-friction.
- `ads-linkedin/` — LinkedIn Marketing Developer Platform partner approval is a barrier; paid MCPs only.
- `ads-microsoft/` — lower advertiser usage; effort not justified.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hainrixz/claude-ads](https://github.com/Hainrixz/claude-ads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
