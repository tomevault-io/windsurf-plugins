---
trigger: always_on
description: >
---


# Corporate Decision Panel

A boardroom in a box. Present any business issue and receive structured,
multi-perspective analysis with engineered dissent -- not consensus from a
single voice, but a decision that shows where expert perspectives collide
and why.

---

## Setup Check

Before executing any command, check if `.claude/agents/ceo.md` exists.

**If missing:** Run `python3 install.py` from the skill directory. Inform the
user that slash commands require a Claude Code restart to become available,
then proceed to execute the current command.

**If present:** Proceed directly to command execution.

---

## Invocation Grammar

### Tier 1 -- Hallway Question
```
/cdp:consult [role] [mode?]: [question]
```
Quick, opinionated consult with one C-suite agent. No CEO, no routing,
no team leads. Produces an **Advisory Note** (3-5 sentences) and an
**Advisory Document** (DOCX memo).

- `/cdp:consult cfo: Can we afford to hire 15 engineers this quarter?`
- `/cdp:consult ciso guardian: What are the risks of this vendor integration?`
- `/cdp:consult vp-sales pioneer: How does this feature help us sell more?`

**Roles:** `ceo`, `coo`, `cfo`, `cto`, `clo`, `ciso`, `cao`, `vp-sales`,
`vp-delivery`, `cso`

### Tier 2 -- Working Session
```
/cdp:panel [roles] [mode?]: [issue]
```
CEO frames and routes to 2-4 C-suite members. Full domain analysis with
team lead delegation. CEO produces lightweight synthesis. Produces a
**Panel Assessment** (~1 page).

- `/cdp:panel finance tech: Should we build this feature in-house?`
- `/cdp:panel pioneer finance tech sales: Should we acquire CompetitorX?`

Production always triggers after the Panel Assessment, producing the same
five artifacts as Tier 3 (HTML, PPTX, DOCX, Results PDF, Capsule PDF)
with proportionally lighter content.

### Tier 3 -- Board Meeting
```
/cdp:deliberate [mode?]: [issue]
```
Full five-phase cascade. All relevant C-suite activated via routing table.
Full team lead analysis. Pre-mortem challenge. Complete CEO deliberation.
Produces a **Decision Record** (3-5 pages). Production always triggered.

- `/cdp:deliberate: Should we pivot to a platform model?`
- `/cdp:deliberate guardian: Should we take on $10M in debt for expansion?`
- `/cdp:deliberate sentinel: Should we acquire CompetitorX?`

### Auto-Triage
```
/cdp:evaluate: [issue]
```
CEO assesses the issue and recommends a tier, mode, and routing. The user
accepts, overrides, or selects a different configuration.

**CEO evaluates:** scope (single/multi/cross-cutting), impact (low through
critical), reversibility (easily reversed through irreversible).

**Output:**
```
ISSUE TRIAGE: [Issue Title]
Scope: [single-domain | multi-domain | cross-cutting]
Impact: [low | medium | high | critical]
Reversibility: [easily reversed | difficult | irreversible]
Recommended Tier: [tier] -- [rationale]
Recommended Mode: [mode] -- [rationale]
Alternative: [mode] -- [what it would reveal]
Scale: ~[N] agents ([K] C-suite x ~[L] team leads avg)
```

### Multi-Mode Syntax
Domain analysis runs once. CEO synthesis runs per mode. Cost: ~1.1x for
up to 5x the strategic insight.

```
/cdp:deliberate guardian vs pioneer: [issue]       # Two-mode comparison
/cdp:deliberate guardian vs analyst vs sentinel: [issue]  # Three modes
/cdp:deliberate all-modes: [issue]                 # All five modes
/cdp:consult cfo guardian: [question]              # Tier 1 with mode
/cdp:panel pioneer finance tech: [issue]           # Tier 2 with mode
```

Multi-mode produces a **Comparative Decision Record** with shared analysis,
per-mode synthesis, divergence analysis, and Mode Sensitivity rating.

### Production Re-run
```
/cdp:production [session-path?]
```

Re-runs only the production pipeline for an existing session using the
persisted `RECORD.md`. Does not re-run the deliberation cascade.

Session resolution:
1. Explicit path → validate it contains `RECORD.md`
2. Slug substring match → scan `.cdp-output/*/RECORD.md`, disambiguate if multiple
3. No argument → most recent session (by date prefix)
4. No sessions → error

Examples:
- `/cdp:production` — most recent session
- `/cdp:production .cdp-output/2026-02-28_should-we-acquire-competitor-x/`
- `/cdp:production acquire-competitor` — fuzzy slug match

### Session Resume
```
/cdp:resume [session-path?]
```

Resumes an interrupted CDP session by detecting how far it progressed and
continuing from that point. Uses the same session resolution rules as
`/cdp:production`. See `config/orchestration-protocol.md` Session Resume
Protocol for detection logic and resume points.

Cannot resume with zero recommendation files (re-run the original command).
Cannot change routing or mode after resume.

- `/cdp:resume` — most recent session
- `/cdp:resume .cdp-output/2026-03-01_should-we-pivot/`
- `/cdp:resume pivot` — fuzzy slug match

### Session Cleanup
```
/cdp:cleanup [--older-than days?]
```
Deletes old CDP session directories from `.cdp-output/` with age-based
filtering and a confirmation prompt before deletion. Default threshold
is 30 days.

- `/cdp:cleanup` -- delete sessions older than 30 days
- `/cdp:cleanup --older-than 7` -- delete sessions older than 7 days

---

## Decision Modes

Five CEO synthesis prompt modifiers. Domain analysis is identical across
modes -- different weighting produces different decisions from the same

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apatheticus/corporate-decision-panel](https://github.com/apatheticus/corporate-decision-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
