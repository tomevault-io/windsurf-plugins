---
trigger: always_on
description: An evidence-graded library of agent-executable thinking-method skills, for AI agents and the humans who work with them. The plugin installs as `thinking-framework-skills`. Skill IDs are namespaced `thinking-framework-skills.<method>`, and installable skill names carry a `think-` prefix (for example `think-premortem`) to avoid cross-plugin collisions. Sibling to `pm-skills`, no technical coupling: `thinking-framework-skills` helps decide *what* to work on and *why* it is sound; `pm-skills` helps 
---

# thinking-framework-skills - agent guide

An evidence-graded library of agent-executable thinking-method skills, for AI agents and the humans who work with them. The plugin installs as `thinking-framework-skills`. Skill IDs are namespaced `thinking-framework-skills.<method>`, and installable skill names carry a `think-` prefix (for example `think-premortem`) to avoid cross-plugin collisions. Sibling to `pm-skills`, no technical coupling: `thinking-framework-skills` helps decide *what* to work on and *why* it is sound; `pm-skills` helps execute *how*.

## What makes a skill here different

Each skill is built around four commitments, not just a prompt:

1. **Mechanism over ritual.** The skill implements the durable cognitive move, named descriptively, not a trademarked brand.
2. **Honest evidence grading.** Every skill carries an evidence tier and an `evidence/dossier.md` that says what the research does and does not support, and flags where evidence is transferred from human studies rather than validated for AI use.
3. **Artifact, not prose.** Every skill emits a concrete, reusable artifact (a risk register, an option matrix, a perspective review).
4. **Explicit "When NOT to Use."** Each skill states where it misleads, to guard against cargo-cult execution.

The canonical statement of all four commitments is in [Philosophy](https://thinking-framework-skills.productonpurpose.com/about/philosophy/) on the docs site.

## Skills

<!-- BEGIN GENERATED SKILLS (scripts/gen-agents.mjs from frameworks/registry.mjs + skills/) - do not hand-edit below this line -->
| Skill | Family | Evidence | Artifact |
|---|---|---|---|
| [`think-analysis-of-competing-hypotheses`](skills/think-analysis-of-competing-hypotheses/SKILL.md) | assumption-and-belief-challenge | X | honest redirect brief |
| [`think-authentic-dissent`](skills/think-authentic-dissent/SKILL.md) | assumption-and-belief-challenge | **S** | dissent audit |
| [`think-consider-the-unknowns`](skills/think-consider-the-unknowns/SKILL.md) | assumption-and-belief-challenge | M | known unknowns ledger |
| [`think-ladder-of-inference-check`](skills/think-ladder-of-inference-check/SKILL.md) | assumption-and-belief-challenge | P | reasoning trace |
| [`think-red-team-light`](skills/think-red-team-light/SKILL.md) | assumption-and-belief-challenge | M | adversarial critique |
| [`think-complexity-domain-sort`](skills/think-complexity-domain-sort/SKILL.md) | decision-and-option-evaluation | C | complexity domain sort with actions |
| [`think-decision-option-review`](skills/think-decision-option-review/SKILL.md) | decision-and-option-evaluation | P | option matrix |
| [`think-dialectical-bootstrapping`](skills/think-dialectical-bootstrapping/SKILL.md) | decision-and-option-evaluation | M | dialectical estimate |
| [`think-eisenhower-moscow-pareto`](skills/think-eisenhower-moscow-pareto/SKILL.md) | decision-and-option-evaluation | P | prioritization preset artifact |
| [`think-expected-value-decision-tree`](skills/think-expected-value-decision-tree/SKILL.md) | decision-and-option-evaluation | P | decision tree ev |
| [`think-fermi-estimation`](skills/think-fermi-estimation/SKILL.md) | decision-and-option-evaluation | M/P | fermi decomposition worksheet |
| [`think-interest-based-negotiation`](skills/think-interest-based-negotiation/SKILL.md) | decision-and-option-evaluation | P | negotiation preparation map |
| [`think-linear-model-aggregation`](skills/think-linear-model-aggregation/SKILL.md) | decision-and-option-evaluation | **S** | scoring model |
| [`think-minimax-regret`](skills/think-minimax-regret/SKILL.md) | decision-and-option-evaluation | P | regret matrix |
| [`think-one-way-vs-two-way-door`](skills/think-one-way-vs-two-way-door/SKILL.md) | decision-and-option-evaluation | P | reversibility classification |
| [`think-pairwise-comparison`](skills/think-pairwise-comparison/SKILL.md) | decision-and-option-evaluation | P | pairwise comparison matrix |
| [`think-what-would-have-to-be-true`](skills/think-what-would-have-to-be-true/SKILL.md) | decision-and-option-evaluation | P | assumption ledger |
| [`think-assumption-reversal`](skills/think-assumption-reversal/SKILL.md) | divergent-ideation | P | assumptions and reversals sheet |
| [`think-brainwriting`](skills/think-brainwriting/SKILL.md) | divergent-ideation | **S** | idea pool |
| [`think-far-analogy-ideation`](skills/think-far-analogy-ideation/SKILL.md) | divergent-ideation | **S** | far analogy transfer sheet |
| [`think-morphological-analysis`](skills/think-morphological-analysis/SKILL.md) | divergent-ideation | P | morphological field |
| [`think-question-burst`](skills/think-question-burst/SKILL.md) | divergent-ideation | P | ranked question set |
| [`think-scamper`](skills/think-scamper/SKILL.md) | divergent-ideation | P | scamper expansion sheet |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [product-on-purpose/thinking-framework-skills](https://github.com/product-on-purpose/thinking-framework-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
