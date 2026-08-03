---
trigger: always_on
description: This file is read automatically by Claude Code (and any Claude-compatible agent) when this repository is loaded as a workspace or plug-in. Anything written here applies to every session that touches the Brand OS.
---

# CLAUDE.md - Instructions for any Claude session loading this Brand OS as a plug-in

This file is read automatically by Claude Code (and any Claude-compatible agent) when this repository is loaded as a workspace or plug-in. Anything written here applies to every session that touches the Brand OS.

If you are a Claude session reading this, the rules below are mandatory, not advisory. The owner of this repo owns the brand voice. Get the voice right.

## First-time setup

If `00-foundations/positioning.md` still contains `{{ YOUR_POSITIONING_LINE }}` placeholders, the Brand OS has not been onboarded yet. Stop and run the wizard before generating anything else:

```bash
python3 tools/onboard.py
```

The wizard fills the canonical foundation files via interactive Q&A (about 25-30 questions, save-as-you-go, safe to interrupt and resume).

## What this repo is

The single source of truth for everything this brand says to a customer. Brand voice, positioning, persuasion canon (Behavioral Economics + Voss/NSTD + Cialdini-Sutherland), funnel architecture, touchpoint copy, evidence, decisions, anti-patterns.

Current facts live on one page: `LAYER0-LIVE.md`. Read it before trusting a fact found in prose anywhere else in the repo. The dated decision files in `06-decisions/` (live statuses in `06-decisions/INDEX.md`) are the ultimate source; any PR that changes a fact must update `LAYER0-LIVE.md` in place in the same PR.

If you are about to write or audit:

- An email or SMS
- A PDP, lander, or hero block
- A social caption (Instagram, TikTok)
- An affiliate brief or creator script
- A founder story or About page
- A confirmation, refund, or customer-service reply
- A press response
- An investor narrative that quotes customer-facing copy

then you are working inside this repo's domain. Read the rules below before generating anything.

## The Marketing Brain - ALWAYS query it first

Before answering any marketing question, drafting any customer copy, or proposing any tactic, invoke the Marketing Brain CLI:

```bash
python3 tools/marketing_brain.py search "<natural-language question>" --top 5
python3 tools/marketing_brain.py explain "<question>"
python3 tools/marketing_brain.py tactic <tactic_name>
python3 tools/marketing_brain.py for-stage <funnel_stage>
python3 tools/marketing_brain.py for-vector <content_vector>
python3 tools/marketing_brain.py icp
```

The Brain wraps four layers, each filtered through the layer above it:

- **Layer 0 - positioning anchors** (ICP + content vectors + Wall 1 + Wall 2 + founder anchor + forbids/licenses lists + voice register refs + never-name list). The strategic frame.
- **Layer 1 - cocktails** in `01-canon/cocktail-recipes.md`. Pre-vetted stacks with Wall-1 / Wall-2 hygiene applied. Use them first.
- **Layer 1.5 - canon principles** in 7 school files: `behavioral-economics.md` (21 BE principles), `nstd-tactics.md` (21 Voss / NSTD tactics), `cialdini-sutherland.md` (22 Cialdini + Sutherland + Ogilvy BSP), `llm-seo-canon.md` (6 LLM SEO and Content Engineering), `dtc-mechanics.md` (8 post-iOS-14 DTC operating mechanics), `subscription-mechanics.md` (5 subscription retention mechanics), `pricing-mechanics.md` (5 premium-pricing mechanics). 88 principles with explicit Where-to-use / Where-NOT guidance.

Optionally a **Layer 2 - raw evidence rows** if you have a third-party research corpus. Drop it as `01-canon/nudge-vault-raw-capture.txt` (one block per `--- ID ---` line) and the Brain auto-indexes it.

The Brain serves three surfaces:

- **CLI** via `python3 tools/marketing_brain.py search "<query>"` (stdlib only, no external API)
- **HTML web interface** via `python3 web/app.py` -> `http://127.0.0.1:8081/` (pages for search / ICP / canon / tactic / vector / stage / guidelines / assets / howto / stats)
- **JSON API** at `/api/search`, `/api/explain`, `/api/icp`, `/api/canon`, `/api/tactic/<name>`, `/api/for-vector/<key>`, `/api/for-stage/<name>`, `/api/list-tactics`, `/api/list-stages`, `/api/stats` for programmatic consumers

Layer 0 is the GATE: any tactic that does not serve at least one of the content vectors AND respect both walls gets rejected. Beneath that gate, priority is cocktail > canon principle > raw evidence row.

The Brain never fabricates. If a query has no hit in the corpus, it returns `no-match` and tells the caller to gather more evidence rather than guessing.

## The seven hard rules

These are filled by the onboarding wizard into `00-foundations/brand-voice.md`. CI enforces them. A PR that violates one fails the Brand Voice Check workflow.

Examples of strong hard rules (replace with yours during onboarding):

- Short hyphens only - no em or en dashes
- No emojis in customer-facing copy
- No exclamation marks in customer-facing copy
- No medicinal vocabulary (food/supplement brands)
- Period-terminated declarative cadence
- Every external claim must trace to a primary source
- No fabricated numbers, dates, customer counts, or studies

Edit `00-foundations/brand-voice.md` and re-run `tools/marketing_brain.py rebuild-index` after onboarding to lock yours in.

## Names and identity


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mishalyalin/brand-os-template](https://github.com/mishalyalin/brand-os-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
