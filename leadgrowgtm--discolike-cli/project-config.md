---
trigger: always_on
description: >
---


# DiscoLike Account List Builder

End-to-end account sourcing workflow. From a seed domain or ICP description to a validated, normalized prospect list ready for Clay enrichment and outbound campaigns.

**This skill is CLI-first.** All DiscoLike operations use the `discolike` CLI (installed via `pip install -e discolike-cli/`). The CLI handles cost tracking, caching (90-day extract, 7-day profiles), budget guardrails, and output formatting automatically.

## Core Rules (Non-Negotiable)

These four rules were learned the hard way. Violating any of them produces bad lists while feeling productive. Read them before touching any phase.

### Rule 1: Semantic overlap trap — never phrase-negate target language

**Target ICP language overlaps with noise language because they serve the same persona.** If you're building a list of marketing SaaS (products sold to marketers), the targets will have "marketing" all over their sites because they talk to marketers. Negate "marketing" and you kill your targets along with the agencies.

The pattern is universal:
- Marketing SaaS targets mention "marketing" (sell to marketers)
- Carbon accounting SaaS targets mention "sustainability consulting" (sell to sustainability teams)
- HR tech targets mention "recruitment" (sell to recruiters)
- Sales enablement SaaS targets mention "outbound" and "SDRs" (sell to sales ops)
- Legal tech targets mention "legal services" (sell to law firms)

The target and the noise share vocabulary because they exist in the same market. Keyword exclusion cannot distinguish "a marketing agency" from "a SaaS that sells to marketing agencies."

**Before proposing any `--negate-phrase`, ask: could a target company's homepage contain this word in a legitimate context? If yes, the exclusion is too broad.** Use semantic classification (DiscoGen or validate) instead — it discriminates by business model, not vocabulary.

**Safe exclusion hierarchy (most safe → least safe):**
1. Domain-level exclusions (exact match, zero overlap risk)
2. Category-level exclusions (NLP-classified, smarter than keywords)
3. Semantic qualification via DiscoGen/validate (the right tool for most exclusions)
4. Phrase exclusions (only when the phrase has zero possible target overlap — rare)

### Rule 2: Inclusion-first — let the data surface the problem

Start every discovery run with positive filters only. No `--negate-phrase`, no `--negate-icp-text`, no `--negate-domain` in the first call. Pull 50 records. Analyze what's actually noise. THEN apply targeted exclusions based on what you observed.

The reason: upfront exclusions are guesses about what will be wrong. They often kill good results silently (a commercial EPC that mentions "residential" once gets dropped). The data itself is a better feedback signal than your priors.

**Autonomous exclusion loop (Phase 4):**

1. **Round 0:** Inclusion-only filters. Pull 50 records. No negations.
2. **Data conversation (up to 4 autonomous iterations):**
   - Analyze batch. Identify dominant noise pattern.
   - Propose ONE targeted exclusion OR ONE semantic classification question as a hypothesis.
   - Re-run discover (or run DiscoGen for classification).
   - Verify the problem is reduced. If yes, lock in that action.
   - If the problem shifted, propose the next hypothesis.
3. **Cap at 4 iterations.** Terminate early if fit rate is at target or diminishing returns hit.
4. **Report to user** with final batch + iteration trail summary. User decides whether to proceed to full pull.

**One prompt per round, one action per round.** Don't try to do multiple things per iteration — it makes the feedback signal muddy.

Each iteration costs ~$0.22 for discover 50. Four iterations = ~$1. Cheap compared to the cost of bad upfront exclusions killing good results.

### Rule 3: Capital signal, not size — don't tight-filter on employees

Employee count is a bad proxy for outbound fit. A 15-person Series A startup with $10M raised is a better outbound buyer than a 100-person bootstrapped consulting firm. We care about capital, not headcount.

Three reasons:
1. **DiscoLike's employee data is weak for small companies** (buckets are coarse, often stale, frequently missing).
2. **Capital ≠ size.** Funded startups and manufacturers with working capital are real buyers regardless of headcount.
3. **Tight filters exclude ideal targets** (well-funded 20-person startups, 40-person commercial EPCs with project financing).

**Use `--employees "1,1500"` as a loose upper cap** (only excludes massive enterprises). Don't set a minimum. Push capital qualification DOWNSTREAM to Clay enrichment where funding data actually lives (Crunchbase, PitchBook, company filings).

**Also important:** don't pre-filter by business model. Consultants with direct B2B sales motions ARE valid targets. Manufacturers with direct commercial sales ARE valid targets. The real filter is "direct B2B motion vs channel/distributor" — answered by DiscoGen during validation, not by upfront assumptions about "SaaS is our sweet spot."

### Rule 4: Realistic benchmarks — 60-70% is the target, not 80%

| Fit rate | Interpretation |
|---|---|
| 60-70% strong fit | **Normal outcome. Proceed.** |
| >70% | Unusually clean list. Usually means inclusions are very tight. |
| 40-60% | Needs refinement. Run another iteration. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeadGrowGTM/discolike-cli](https://github.com/LeadGrowGTM/discolike-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
