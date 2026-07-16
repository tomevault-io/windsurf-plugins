---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PM diagnostic tool with 2 complementary loops for SEA ecommerce PMs across all KPIs and markets (SG, MY, AU, NZ, PH, HK, TH, ID):

**Intelligence Loop (Phase 1 — active):** 4-step diagnostic loop.
1. **Signal** — query configured Domo KPI datasets; surface which metrics moved, where, by how much
2. **Diagnose** — two parallel sub-steps: (2a) Domo feedback datasets for corroborating voice-of-customer evidence; (2b) independent code review to localise the failure and map mechanisms
3. **Hypothesise** — convert the favored diagnosis into rival-tested, falsifiable hypotheses and A/B experiment designs
4. **Prioritise** — score and rank experiments by confidence × impact × scope; PM approves which advance

**Inspiration Loop (active — parallel to Phase 1):** PM-triggered ideation loop that produces prototype bets.
1. **Scout** — load Agent 10 signals (what's broken) + browse SEA frontends (what we have today) + scoped market scan (what's possible)
2. **Gate 1** — PM confirms pre-mortem scenario and prototype idea; bet recorded with target metric and odds
3. **Classify** — bet classified by type (pain / shine / pain+shine); drives downstream loop investment
4. Downstream: prototype builder → launch validator → fit tracker (agents 22–24, in design)

Both loops feed Agent 13: the Intelligence Loop provides a diagnosis artifact plus code-grounded experiment designs; the Inspiration Loop provides market context and PM odds as optional enrichment for scoring and tiebreaking.

Every gate is human-approved. Domo evidence absence is a hard stop — the system halts and asks, it does not infer. Phase 2 adds the action layer (GitHub effort estimation + Jira story creation) once Phase 1 is proven.

## Operating Model

This repo uses a 3-layer architecture that separates concerns to maximise reliability. LLMs are probabilistic; business logic is deterministic. This system fixes that mismatch.

### 3-Layer Architecture

**Layer 1 — Directive (What to do)**
SOPs written in Markdown, living in `directives/` and agent `SKILL.md` files. Define goals, inputs, tools/scripts to use, outputs, and edge cases. Natural language instructions, like you'd give a mid-level employee.

**Layer 2 — Orchestration (Decision making)**
This is Claude. Job: intelligent routing. Read directives, call execution tools in the right order, handle errors, ask for clarification, update directives with learnings. Claude is the glue between intent and execution — don't try to do the work manually, read the directive and run the right script.

**Layer 3 — Execution (Doing the work)**
Deterministic scripts in `execution/` (Python) and `scripts/` (Node). Handle API calls, data processing, file operations. Reliable, testable, fast. Use scripts instead of manual work.

**Why this works:** if Claude does everything itself, errors compound. 90% accuracy per step = 59% success over 5 steps. Push complexity into deterministic code so Claude can focus on decision-making.

### Operating Principles

**1. Check for tools first**
Before writing a script, check `execution/` per the relevant directive. Only create new scripts if none exist.

**2. Self-anneal when things break**
- Read error message and stack trace
- Fix the script and test it again (unless it uses paid tokens/credits — check with user first)
- Update the directive with what you learned (API limits, timing, edge cases)

**3. Update directives as you learn**
Directives are living documents. When you discover API constraints, better approaches, common errors, or timing expectations — update the directive. Don't create or overwrite directives without asking unless explicitly told to.

**4. Optimise for context window**
- Apply `directives/context_window_budget.md` on all multi-step runs
- Load minimum scope first (directive + script + active outputs), then expand only when blocked
- Prefer deterministic summarisation in `execution/` over long narrative reasoning
- Avoid replaying unchanged history; reference prior artifacts and record only deltas

### Self-annealing loop

Errors are learning opportunities. When something breaks:
1. Fix it
2. Update the tool
3. Test the tool, make sure it works
4. Update directive to include the new flow
5. System is now stronger

### File Organisation

**Deliverables vs Intermediates:**
- **Deliverables**: Confluence pages, Google Sheets/Slides, or other cloud outputs the user can access
- **Intermediates**: Temporary files needed during processing

**Directory structure:**
- `.tmp/` — All intermediate files (scraped data, temp exports). Never commit, always regenerated.
- `execution/` — Python scripts (deterministic tools)
- `directives/` — SOPs in Markdown (the instruction set)
- `scripts/` — Node.js scripts (e.g. Puppeteer capture)
- `.env` — Environment variables and API keys (git-ignored)
- `outputs/` — Agent-generated files (operational data)

**Key principle:** Local files are only for processing. Deliverables live in cloud services (Confluence, Google Sheets, etc.) where the user can access them. Everything in `.tmp/` can be deleted and regenerated.

### Cloud Webhooks (Modal)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lilyydavid/ai-diagnostic-loop-mirror](https://github.com/lilyydavid/ai-diagnostic-loop-mirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
