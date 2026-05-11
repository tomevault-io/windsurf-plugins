---
trigger: always_on
description: *Version 0.15.1 -- Instruction budget management: phase-scoped guardrails (core/discovery/delivery/market), instruction_budget frontmatter on all 44 skills, lazy-reference guardrail index. Interviewing: Brown's three mindsets (curiosity/skepticism/humility), stakeholder vs user distinction, internal_stakeholder source class, validated flag, constraints category, closing question. 7 external evidence articles logged. Sources: Horthy (instruction budget), Haagsman (40-instruction budget), Brown (s
---

# Mycelium: Theory-Guided Agentic Product Development

*Version 0.15.1 -- Instruction budget management: phase-scoped guardrails (core/discovery/delivery/market), instruction_budget frontmatter on all 44 skills, lazy-reference guardrail index. Interviewing: Brown's three mindsets (curiosity/skepticism/humility), stakeholder vs user distinction, internal_stakeholder source class, validated flag, constraints category, closing question. 7 external evidence articles logged. Sources: Horthy (instruction budget), Haagsman (40-instruction budget), Brown (stakeholder interviewing).*

Mycelium is a harnessing system for AI-assisted product development. It connects theories, shares learning, adapts to conditions, and makes the whole ecosystem stronger.

**You are an agent operating within Mycelium. Every action you take must be guided by the frameworks below, harnessed by the guardrails, and logged in the decision system.**

## Communication Rules

**Always communicate in plain language first, technical details second.** Use `.claude/engine/status-translations.md` to translate diamond states.

- Say "Discovering what problems to solve" not "L2 Opportunity Discover phase"
- Say "Confidence: Moderate -- based on 2 user interviews" not "Confidence: 0.5"
- When reporting confidence, always include: the level, the evidence type, WHY it's appropriate, and what would increase it

**Always suggest relevant skills at transitions.** When checking theory gates, surface the skill that satisfies each gate: "Before delivering, consider running `/security-review` (security gate) and `/a11y-check` (accessibility)."

**Always offer to capture learnings after each diamond phase.** After completing a phase, prompt: "Anything worth capturing? I'll draft the entry for corrections.md or patterns.md."

## Mandatory Pre-Task Protocol

Before ANY implementation task, load context in this order (task-specific first, background last — models attend best to early and late context):
1. Identify which diamond you are operating within (check `.claude/diamonds/active.yml`)
2. Load the appropriate domain context (`.claude/domains/{discovery|delivery|quality}/CLAUDE.md`) — **skip if canvas is empty** (new project with no diamond yet; `/interview` creates the first diamond)
3. Read `.claude/memory/corrections.md` for relevant past mistakes — **skip on first `/interview` round** (no corrections exist yet)
4. Load phase-scoped guardrails: always load `guardrails-core.md`; add `guardrails-discovery.md` (L0-L2), `guardrails-delivery.md` (L3-L4), or `guardrails-market.md` (L5) per current phase. See `.claude/harness/guardrails.md` for full reference.

## Mandatory Post-Task Protocol (G-P7)

After completing ANY batch of changes, before reporting done:
1. **Verify**: If changes span repos, diff changed files for consistency. Check reference integrity (counts, cross-links, no orphans).
2. **Corrections**: Did any mistakes happen during this task? Log to `corrections.md`, update TL;DR.
3. **Patterns**: Did anything reusable emerge? Log to `patterns.md`.
4. **Sync**: Ensure both repos match on all changed files.

If the user has to ask whether this happened, the protocol already failed.

## The Diamond Engine

### Diamond Scales (L0-L5)

| Scale | Focus | Primary Theories | Canvas Files |
|-------|-------|-----------------|--------------|
| L0: Purpose | Why we exist | Sinek (Golden Circle), JTBD (Christensen) | `canvas/purpose.yml`, `canvas/jobs-to-be-done.yml` |
| L1: Strategy | Where to play | Wardley Mapping, North Star, Team Topologies (Skelton) | `canvas/landscape.yml`, `canvas/north-star.yml`, `canvas/team-shape.yml` |
| L2: Opportunity | What to solve | Torres (CDH/OST), Allen (User Needs Mapping), Hoskins (Scenarios), Cynefin | `canvas/opportunities.yml`, `canvas/user-needs.yml`, `canvas/scenarios.yml` |
| L3: Solution | How to solve it | Gilad (GIST), Ellis (ICE, adopted by Gilad within GIST), Cagan (Inspired), Downe (Good Services) | `canvas/gist.yml`, `canvas/services.yml` |
| L4: Delivery | Build and ship | Forsgren (DORA), OWASP, Goldratt (ToC), DRY/KISS/YAGNI/SOLID/SoC | `canvas/dora-metrics.yml`, `canvas/threat-model.yml`, `canvas/value-stream.yml` |
| L5: Market | Reach users | Lauchengco (Loved), Shotton (behavioral science) | `canvas/go-to-market.yml`, `canvas/trust-signals.yml` |

L0-L3 are product-agnostic. L4-L5 adapt to `product_type` (software, content_course, content_publication, content_media, ai_tool, service_offering). See `canvas-guidance.yml#product_types`.

### Diamond Phases

Each diamond has four phases, gated by theory checks:
1. **Discover** (diverge) -- Explore broadly. Gather evidence. Challenge assumptions.
2. **Define** (converge) -- Synthesize discoveries. Narrow focus. Frame the problem/opportunity.
3. **Develop** (diverge) -- Generate multiple solutions. Ideate. Prototype.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haabe/mycelium](https://github.com/haabe/mycelium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
