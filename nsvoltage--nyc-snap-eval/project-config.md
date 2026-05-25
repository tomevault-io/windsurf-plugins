---
trigger: always_on
description: Operating manual for Claude Code on this repository. Read this first, every session.
---

# CLAUDE.md

Operating manual for Claude Code on this repository. Read this first, every session.

## What we are building

A statute-grounded evaluation suite for caseworker-facing SNAP benefits-navigation AI in New York, plus a narrow reference implementation. The artifact is meant to look like an Anthropic research release — published grader, validated judge, honest limitations, generous credit, clear scope. The audience is a hiring manager on Anthropic's Beneficial Deployments team.

The system the eval is anchored to — NYC's MyCity chatbot — was discontinued by the Mamdani administration on 2026-02-05. Category A is therefore a post-mortem regression set rather than a live A/B comparator; see [ADR-008](docs/decisions/ADR-008-post-obbba-post-mycity-amendments.md) §4. The eval's value is reinforced by the shutdown, not undercut by it — the new administration's decision is consistent with the project's thesis that a measurable bar should have been applied before deployment.

## The single most important thing

**This is an illustrative suite, not a benchmark.** Approximately seventy cases across seven categories, scoped to one program in one jurisdiction at one point in time. Every design choice should serve that scope. If a decision would push us toward "comprehensive coverage of all SNAP eligibility," push back — that's not what we're building. We are demonstrating a way of thinking about responsible evaluation, with a working implementation as proof the eval is well-formed.

## The four documents that govern this work

In order of precedence when guidance conflicts:

1. **`docs/conventions.md`** — code style, voice, file layout. Hard rules.
2. **`docs/project-plan.md`** — the v2 build plan, day-by-day. What ships in v1, what is forward work.
3. **`docs/decisions/`** — Architecture Decision Records. Why we chose Promptfoo over MLflow, why caseworker-facing over applicant-facing, etc. These are the answers to "but what about X."
4. **`docs/research/`** — the four upstream research reports. Reference material for design judgment calls. Cite them when relevant, don't repeat them.

If you find yourself about to deviate from any of these, stop and ask the human.

## How we work together

The human is the HITL — they make the calls on scope, voice, and trade-offs. Your job is to execute, surface decisions cleanly, and flag drift. Specifically:

- **Surface before deciding.** When you hit a fork — should this be a CSV or YAML? Should the judge be Sonnet or Opus? — ask. Don't pick the more impressive option without surfacing the choice.
- **Default to the simpler option.** Propel's entire infrastructure is 14 lines of YAML and a Google Sheet. We are not building Propel + complexity. We are building Propel + statute grounding + judge transparency. That's it.
- **Match the voice in `docs/conventions.md`.** No marketing language. No "we're excited to announce." No emoji. No superlatives about our own work. Read the voice section before writing any prose.
- **Cite specific files when relevant.** "Per ADR-003" or "per `docs/research/r3-synthetic-data.md` §4" is better than vague gestures at the plan.
- **Don't expand scope.** If a task is "author the MyCity replay cases," don't also refactor the eval harness. Surface the refactor as a separate suggestion.

## Hard rules

These do not bend. If a request would violate one, push back before acting.

1. **Every eval case must have a `citation` field that resolves to a public source.** No exceptions. If we can't cite it, we don't test it. See `docs/conventions.md` §2.
2. **The judge model is pinned and the meta-prompt is published.** Claude Opus 4.5 grades; the full prompt lives at `eval/graders/llm_judge_meta_prompt.md` and changes go through ADRs.
3. **Eligibility math is never computed by the LLM.** The reference implementation calls the NYC Benefits Screening API or a deterministic Python port. The LLM narrates the result. See ADR-002.
4. **No PII-realistic synthetic data.** Personas are composite archetypes drawn from cited demographic research. Ranges and categories, not single specific values that read like a real person. See `docs/research/r3-synthetic-data.md` §"Synthetic persona generation."
5. **The simulated baseline is labeled as a simulation, every time it's referenced.** It is not a reproduction of any deployed system. See ADR-004.
6. **Limitations are first-class.** Every claim has a paired limitation. The README, methodology PDF, and blog post each have explicit limitations sections enumerating at least five specific constraints. See `docs/conventions.md` §"Voice."
7. **Attribution is in the first 1,000 words of any public-facing document.** Propel, The Markup, GOV.UK GDS, Nava Labs, Code for America, Stanford RegLab. Named, with URLs. Not in a footer.

## Where to find things

- **The plan, in order:** `docs/project-plan.md` has the day-by-day sequencing.
- **Why we chose what we chose:** `docs/decisions/` — ADRs are short and dated, read the headers to find the relevant one.
- **The four research reports** (these are the upstream synthesis we built the plan from):
  - `docs/research/r1-nyc-poverty-and-benefits-infrastructure.md` — the demographic and infrastructure context for why this work matters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NSvoltage/nyc-snap-eval](https://github.com/NSvoltage/nyc-snap-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
