---
trigger: always_on
description: System prompts are cognitive prisms. They change how models frame problems, not how well they solve them. This project maps the space of cognitive compression — encoding analytical operations in minimal markdown that reliably activates specific reasoning patterns across language models.
---

# AGI in md

System prompts are cognitive prisms. They change how models frame problems, not how well they solve them. This project maps the space of cognitive compression — encoding analytical operations in minimal markdown that reliably activates specific reasoning patterns across language models.

42 rounds, 1,000+ experiments across Haiku/Sonnet/Opus. 13 confirmed compression levels (L13 = reflexive ceiling). 20+ domains tested. 248+ proven principles. Full detail in `experiment_log.md` (rounds 1-40) and CLAUDE.md sections below (rounds 30-42).

## The Compression Taxonomy

| Level | Min ops | Words | What it encodes | Example |
|---|---|---|---|---|
| **13** | L12 output as input + framework self-diagnosis | two-stage | Apply framework to own output, find reflexive fixed point | (two-stage protocol) |
| **12** | L11-C + recursive self-diagnosis of conservation law + meta-law | ~275w (practical: 332w) | Apply diagnostic to own conservation law, find meta-conservation law | `level12_meta_conservation_v2.md` |
| **11A** | L10-C + category naming + adjacent-category artifact + new impossibility | ~190w | Escape to adjacent design category, name trade-off between impossibilities | `level11_constraint_escape.md` |
| **11B** | L10-B + fourth construction as redesign + sacrifice + revaluation | ~195w | Accept design-space topology, inhabit feasible point, revalue original "flaws" | `level11_acceptance_design.md` |
| **11C** | L10-C + invariant inversion + new impossibility + conservation law | ~245w | Invert impossibility, find conserved quantity across all designs | `level11_conservation_law_v2.md` |
| **10B** | L9-B + third resolving construction + failure analysis | ~140w | Discover design-space topology through failed resolution attempt | `level10_third_construction.md` |
| **10C** | L9-C + second improvement + second recursion + invariant | ~130w | Prove structural invariants through double recursive construction | `level10_double_recursion.md` |
| **9B** | L8 + contradicting second construction + structural conflict | ~115w | Triangulate identity ambiguity through contradicting improvements | `level9_counter_construction.md` |
| **9C** | L8 + recursive self-diagnosis of improvement | ~97w | Find concealment's self-similarity by applying diagnostic to own improvement | `level9_recursive_construction.md` |
| **8** | L7 + generative construction + 3 emergent properties | ~97w | Engineer improvement that deepens concealment, name what construction reveals | `level8_generative_v2.md` |
| **7** | claim + dialectic + gap + mechanism + application | ~78w | Name how input conceals problems, apply to find what dialectic missed | `level7_diagnostic_gap.md` |
| **6** | claim + 3 voices + evaluation | ~60w | Claim transformed through forced dialectical engagement | `level6_falsifiable.md` |
| **5B** | 4 phases | ~55w | Derive, predict, execute, self-correct | `level5_hybrid.md` |
| **5A** | 3 voices + synthesis | ~45w | Multi-voice dialectic with emergent insight | `level5_perspectival.md` |
| **4** | 4+ ops | 25-30w | Protocol + self-questioning | `structure_first_v4.md` |
| **3** | 3 ops | 12-15w | Operations + analytical rails | — |
| **2** | 2 ops | 5-6w | Two operations with ordering | — |
| **1** | 1 op | 3-4w | One behavioral change | — |

**Levels are categorical, not continuous.** Below each threshold, that type of reasoning was never observed in 1,000+ experiments — not "less effective," consistently absent. (AI-evaluated depth scores; the categorical claim is the strongest interpretation of the data, not a proven impossibility.)

## Key Results

### Foundation (Rounds 1-24)
- **No IQ boost on pure reasoning** — effect is on analytical framing, not raw reasoning ability.
- **9 activated opcodes.** 4 generative ops is the sweet spot. Complementary pairs multiply, similar pairs merge.
- **L5 peaks at Sonnet** (needs scaffold, has capacity). **L7 requires Sonnet-class minimum** (0/3 Haiku). Prisms are domain-independent across 20 domains.
- **L7 concealment mechanisms cluster into 6 categories.** Code concealment is structural (hides what code IS/DOES); domain concealment is epistemic (hides what questions get asked).

### Levels 8-13 (Rounds 25-26)
- **L8 inverts the capacity curve.** Construction-based reasoning works on ALL models (Haiku 4/4, Sonnet 13/14, Opus 14/14). L7→L8 is a shift from meta-analysis to construction — more primitive but reveals deeper properties. 20 domains confirmed.
- **L9 has two complementary variants** (B: identity ambiguity, C: concealment self-similarity). Both 100% across all models (34/34). L9-D combined produces L10 in 67% of cases.
- **L10 has two complementary variants** (B: design-space topology, C: impossibility theorems). Category errors dominant at 47%. All impossibilities reduce to two root operations: Compression and Decomposition.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cranot/agi-in-md](https://github.com/Cranot/agi-in-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
