---
trigger: always_on
description: Pre-launch simulation engine: generates research-grounded personas and simulates them through product studies. See [README.md](README.md) for user-facing docs, positioning, and limitations.
---

# Facet — Codebase Guide

Pre-launch simulation engine: generates research-grounded personas and simulates them through product studies. See [README.md](README.md) for user-facing docs, positioning, and limitations.

## Architecture

Six-command pipeline:

```
sim.sh init:       Plan (Opus) → Generate (Sonnet, parallel waves)
sim.sh study:      Simulate (Sonnet, parallel) → Analyze (Opus)    [--runs N for stability]
sim.sh synthesize: Cross-Synthesis (Opus) — unified findings across studies
sim.sh run:        Full lifecycle — init panel + all studies + synthesize (single command)
sim.sh compare:    Diff findings between two panels
sim.sh status:     Panel progress, study completion, cross-synthesis readiness
```

Primary interface:
- **sim.sh** — bash orchestrator for terminal/CI use. Each phase is a `claude --print` subprocess.

Key design: personas are generated ONCE and reused across multiple studies. The `study` subcommand reads a single study config containing all studies and runs the full pipeline.

### Model Routing

| Phase | Model | Why |
|-------|-------|-----|
| Plan | default (Opus) | Reasoning-heavy: segment design, constraint vectors, diversity matrix |
| Generate | `--model sonnet` | Cost efficiency at scale (50 parallel personas) |
| Simulate | `--model sonnet` | Cost efficiency at scale (50 parallel simulations) |
| Analyze | default (Opus) | Synthesis quality: reads all personas + simulations, produces recommendations |
| Cross-Synthesize | default (Opus) | Reads all per-study syntheses + persona summaries, produces unified findings |

### Tool Restrictions

All `claude` invocations are restricted to `Read,Write,Glob,Grep`. No Bash — no escape hatch. The one exception: calibration directory mode adds Glob and Grep to persona generation (normally Read,Write only).

## Wave-Based Generation

Personas are generated in waves of 5 to prevent homogeneity.

```
Wave 1: personas 1-5   → no diversity context
Wave 2: personas 6-10  → sees summaries of 1-5
Wave 3: personas 11-15 → sees summaries of 1-10
...
```

After each wave completes, `extract_persona_summary()` in sim.sh reads each generated persona and builds a one-line summary (name, segment, key traits). This summary block is injected into the next wave's prompt with explicit instructions: "your persona must sound, think, and decide differently from ALL of the above."

Each wave must fully complete before the next starts — the diversity context depends on it.

## Template Patterns

All four templates (`plan.md`, `persona.md`, `simulation.md`, `analysis.md`) follow consistent patterns:

1. **Integrity rules** — anti-sycophancy guardrails. Every template explicitly permits rejection, skepticism, indifference. "This persona is NOT obligated to like the product."
2. **Quality bar** — specific > generic. "$38,500/year" not "moderate salary." Named neighborhoods, not "urban area."
3. **Thinking steps** — brainstorm/analysis sections marked "do NOT include in output." These prompt Claude to reason before writing but keep the output clean.
4. **Consistency self-check** — templates end with verification that stated facts, numbers, and decisions are internally consistent.

### Study-Type Rules

Each study type in `study-types/` specifies which behavioral economics frameworks apply:

| Study Type | Frameworks | Key Metrics |
|-----------|------------|-------------|
| `pricing.md` | Prospect theory, mental accounting, loss aversion, flat-rate bias, zero-price effect | Signup decision, 12-month usage table, renewal, NPS, referral |
| `copy.md` | ELM (central/peripheral routes), construal level, reactance, framing effects | Clarity, trust, motivation, shareability (0-10 each) |
| `features.md` | Kano model (must-be/performance/attractive/indifferent), feature interaction | Per-feature importance, excitement, WTP delta, usage frequency |
| `onboarding.md` | Endowment effect, IKEA effect, psychological ownership, status quo bias, default effect, Fogg B=MAP | Completion funnel, time-to-value, ownership score, status quo shift rate |
| `retention.md` | Hedonic adaptation, sunk cost, peak-end rule, post-purchase rationalization, trust decay | 12-month satisfaction arc, churn taxonomy, honest vs. passive retention rate |
| `custom.md` | None (persona uses natural decision-making) | Gut reaction, reasoning, verdict, concerns per option |

Each study type also has outcome requirements (e.g., "at least 1 persona should churn," "features study is the weakest use case — surface reactions, not ranked lists"). The custom study type has no framework injection — for research questions that don't fit the standard types.

## Template Version Locking

Templates are copied into `.templates/` directories at init and study time:
- `{panel}/.templates/` gets `plan.md` + `persona.md` + `cross-synthesis.md` + `comparison.md` at init
- `{panel}/studies/{study}/.templates/` gets `simulation.md` + `analysis.md` + `stability.md` + `{study-type}.md` at study time

This means existing studies are reproducible even if source templates change. Contributors modifying templates don't break past studies.

## Calibration Data


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saraswatayu/facet](https://github.com/saraswatayu/facet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
