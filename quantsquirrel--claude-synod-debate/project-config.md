---
trigger: always_on
description: **Project:** Multi-agent deliberation system for Claude Code
---

# Synod v3.3 - Agent Architecture

**Project:** Multi-agent deliberation system for Claude Code
**Version:** 3.3.0
**Repository:** https://github.com/quantsquirrel/claude-synod-debate
**License:** MIT

Synod orchestrates Claude, Gemini, and OpenAI in a structured judicial debate (Solver, Critic, Defense, Synthesis) to produce higher-quality decisions than any single model. Implements SID confidence scoring, CortexDebate trust scores, Free-MAD anti-conformity, and ReConcile convergence.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude Code (Entry Point)                 │
│                    /synod [mode] <prompt>                    │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│                    SYNOD ORCHESTRATOR                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  VALIDATOR   │  │  ARCHITECT   │  │  EXPLORER    │      │
│  │  (Claude)    │  │  (Gemini)    │  │  (OpenAI)    │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │              │
│    Phase 0: Classification + Setup                          │
│    Phase 1: Solver Round (Parallel)                         │
│    Phase 2: Critic Round (Trust Scores)                     │
│    Phase 3: Defense Round (Court Model)                     │
│    Phase 4: Synthesis (Final Output + Quality Metrics)      │
└─────────────────────────────────────────────────────────────┘
```

---

## Agent Roles

| Agent | Model | Role | Solver Focus | Defense Role |
|-------|-------|------|-------------|-------------|
| Claude | Sonnet | Orchestrator + Judge | Correctness validation | Neutral arbiter |
| Gemini | flash/pro | Architect | Patterns, structure | Defense counsel |
| OpenAI | gpt-4o/o3/gpt-5.4/gpt-5-mini | Explorer | Edge cases, alternatives | Prosecution |

Phase flow details: `skills/synod/modules/synod-phase{0-4}-*.md`

---

## Phase Flow

1. **Phase 0 - Setup** (`synod-phase0-setup.md`): Classify problem type (coding/math/creative/general), determine complexity, select models, create session. Confidence-based tier promotion when classifier confidence is low.
2. **Phase 1 - Solver** (`synod-phase1-solver.md`): Parallel execution of all agents. SID format required (confidence + semantic_focus). Early exit if all agents confident (>=90).
3. **Phase 2 - Critic** (`synod-phase2-critic.md`): Cross-validation, trust score calculation (CRIS), contention identification. Free-MAD anti-conformity hints for low-confidence models.
4. **Phase 3 - Defense** (`synod-phase3-defense.md`): Adversarial court model. Gemini defends strongest solution; OpenAI prosecutes. Claude judges.
5. **Phase 4 - Synthesis** (`synod-phase4-synthesis.md`): Weighted confidence calculation, mode-specific output, debate quality metrics summary.

Error handling and fallbacks: `skills/synod/modules/synod-error-handling.md`
Session resume protocol: `skills/synod/modules/synod-resume.md`

---

## Core Algorithms

### CRIS Trust Score (CortexDebate)

```
T = min((C x R x I) / S, 2.0)

C = Credibility (0-1): Evidence quality
R = Reliability (0-1): Logical consistency
I = Intimacy (0-1): Relevance to problem
S = Self-Orientation (0.1-1): Bias detection (lower = better)
```

| Rating | Score Range | Action |
|--------|-----------|--------|
| High | T >= 1.5 | Primary source |
| Good | T >= 1.0 | Include normally |
| Acceptable | T >= 0.5 | Include with caution |
| Low | T < 0.5 | Consider excluding |

Confidence intervals and weighted consensus: `tools/synod-parser.py` (`calculate_confidence_interval`, `weighted_consensus`)

### SID Format (Self-Signals Driven)

```xml
<confidence score="[0-100]">
  <evidence>[What supports this?]</evidence>
  <logic>[Reasoning soundness?]</logic>
  <expertise>[Domain confidence?]</expertise>
  <can_exit>[true if confidence >= 90 AND complete]</can_exit>
</confidence>

<semantic_focus>
1. [PRIMARY claim]
2. [SECONDARY claim]
3. [TERTIARY claim]
</semantic_focus>
```

Malformed responses get defaults (C=50, can_exit=false) via `apply_defaults()`.

### Debate Quality Metrics

`parse_response()` emits per-response metrics: response_length, format_compliance, confidence_score, semantic_focus_count, has_evidence, has_logic, has_code. Aggregated via `collect_round_metrics()` and displayed in Phase 4 via `format_metrics_summary()`.

### Confidence-to-Tier Promotion

`get_tier(complexity, confidence)` promotes queries one tier level when classifier confidence is below the `low_confidence` threshold (default 50%). E.g., simple+low_confidence -> standard tier.

---

## Directory Structure

```
claude-synod-debate/
├── AGENTS.md                        # This file
├── CHANGELOG.md
├── README.md
├── plugin.json                      # Plugin metadata
├── marketplace.json                 # Marketplace listing
├── pyproject.toml                   # Python config + dependencies
├── config/
│   ├── synod-modes.yaml             # 5 modes + keywords + thresholds
│   └── synod-templates.yaml         # Mode output templates
├── skills/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quantsquirrel/claude-synod-debate](https://github.com/quantsquirrel/claude-synod-debate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
