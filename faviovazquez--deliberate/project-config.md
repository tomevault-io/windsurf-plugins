---
trigger: always_on
description: Multi-agent deliberation skill. Forces structured disagreement to surface blind spots. Use when making complex decisions, evaluating trade-offs, or when a single confident answer might hide real risks. Invoke with /deliberate or @deliberate followed by your question.
---


# deliberate -- Multi-Agent Deliberation Protocol

**Agreement is a bug.** This skill forces multiple agents to disagree before they agree, surfacing blind spots that single-perspective answers hide.

## Invocation

```
/deliberate "should we migrate from REST to GraphQL?"
/deliberate --full "is this acquisition worth pursuing at 8x revenue?"
/deliberate --quick "monorepo or polyrepo?"
/deliberate --duo assumption-breaker,pragmatic-builder "should we rewrite the auth layer?"
/deliberate --triad architecture "should we split the monolith now?"
/deliberate --triad decision "build vs buy for the notification system"
/deliberate --members assumption-breaker,first-principles,bias-detector "why does our cache keep failing?"
/deliberate --brainstorm "how should we redesign the onboarding flow?"
/deliberate --profile exploration "what's the right approach to AI safety for our product?"
```

## Flags

| Flag | Effect |
|------|--------|
| (no flag) | Auto-detect domain from question, select matching triad |
| `--full` | Convene all 14 core agents. 3-round protocol. |
| `--quick` | Auto-detect triad. 2-round protocol (skip cross-examination). |
| `--duo agent1,agent2` | Dialectic mode. 2 agents, 2 rounds of exchange, then synthesis. |
| `--triad {domain}` | Use pre-defined triad for domain. 3-round protocol. |
| `--members a,b,c,...` | Custom agent selection (2-14 agents). 3-round protocol. |
| `--brainstorm` | Brainstorm mode. See BRAINSTORM.md for full protocol. |
| `--profile {name}` | Use named profile (full, lean, exploration, execution). |
| `--visual` | Launch visual companion for this session. |
| `--save {slug}` | Override auto-generated filename slug for output. |
| `--research` | Run a grounding phase before Round 1: scan the codebase and/or search the web. Agents reason from retrieved evidence, not parametric knowledge alone. **Opt-in only — never the default.** See `protocols/research-grounding.md`. |
| `--research=web` | Web search only (no codebase scan). |
| `--research=code` | Codebase scan only (no web search). |

## The 14 Core Agents

| # | Agent | Function | Tier |
|---|-------|----------|------|
| 1 | `assumption-breaker` | Destroys hidden premises, tests by contradiction, dialectical questioning | high |
| 2 | `first-principles` | Bottom-up derivation, refuses unexplained complexity | mid |
| 3 | `classifier` | Taxonomic structure, category errors, four-cause analysis | mid |
| 4 | `formal-verifier` | Computational skeleton, mechanization boundaries, abstraction | mid |
| 5 | `bias-detector` | Cognitive bias detection, pre-mortem, de-biasing interventions | high |
| 6 | `systems-thinker` | Feedback loops, leverage points, unintended consequences | mid |
| 7 | `resilience-anchor` | Control vs acceptance, moral clarity, anti-panic grounding | mid |
| 8 | `adversarial-strategist` | Terrain reading, competitive dynamics, strategic timing | mid |
| 9 | `emergence-reader` | Non-action, subtraction, intervention audit, minimum intervention | high |
| 10 | `incentive-mapper` | Power dynamics, actor incentives, principal-agent problems | mid |
| 11 | `pragmatic-builder` | Ship it, maintenance cost, over-engineering detection | mid |
| 12 | `reframer` | Dissolves false problems, frame audit, false dichotomies | high |
| 13 | `risk-analyst` | Antifragility, tail risk, fragility profile, barbell strategy | high |
| 14 | `inverter` | Multi-model reasoning, inversion, opportunity cost, cross-domain | mid |

## Optional Specialists

Activated only when their domain-specific triad is selected:

| Agent | Function | Triads |
|-------|----------|--------|
| `ml-intuition` | Neural net intuition, training dynamics, jagged frontier | ai, ai-product |
| `safety-frontier` | Scaling dynamics, capability-safety frontier, phase transitions | ai |
| `design-lens` | User-centered design, honesty audit, "less but better" | design, ai-product |

## Polarity Pairs

These agents are structural counterweights. When both are present, genuine disagreement is almost guaranteed:

| Pair | Tension |
|------|---------|
| assumption-breaker vs first-principles | Top-down destruction vs bottom-up construction |
| classifier vs emergence-reader | Impose structure vs let it emerge |
| adversarial-strategist vs resilience-anchor | Win externally vs govern internally |
| formal-verifier vs incentive-mapper | Abstract purity vs messy human reality |
| pragmatic-builder vs reframer | Ship it vs does it need to exist? |
| pragmatic-builder vs systems-thinker | Fix the bug vs redesign the system |
| risk-analyst vs ml-intuition | Tail paranoia vs empirical iteration |

## Pre-defined Triads

| Domain | Agents | Reasoning Chain |
|--------|--------|-----------------|
| architecture | classifier + formal-verifier + first-principles | categorize -> formalize -> simplicity-test |
| strategy | adversarial-strategist + incentive-mapper + resilience-anchor | terrain -> incentives -> moral grounding |
| ethics | resilience-anchor + assumption-breaker + emergence-reader | duty -> questioning -> natural order |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FavioVazquez/deliberate](https://github.com/FavioVazquez/deliberate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
