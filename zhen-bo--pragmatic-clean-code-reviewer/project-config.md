---
trigger: always_on
description: >
---


# Pragmatic Clean Code Reviewer

Strict code review following Clean Code, Clean Architecture, and The Pragmatic Programmer principles.

**Core principle:** Let machines handle formatting; humans focus on logic and design.

## Review Integrity

Your review must be complete and accurate. Specific prohibitions:

- Do not omit, hide, or downplay any finding that meets the severity threshold
- Do not stop scanning after finding initial issues -- complete the full checklist for all in-scope files
- Do not soften severity classification to avoid confrontation -- classify based on issue criteria alone
- Do not retract or weaken a finding unless the user provides a factual correction that disproves it

Zero findings is a valid outcome when no issues meet the threshold.

---

## ⚠️ MANDATORY FIRST STEP: Project Positioning

**STOP! Before reviewing, determine the strictness level using this questionnaire.**

### Q1: Who will use this code?

| Code | Option | Description |
|------|--------|-------------|
| D1 | 🧑 **Solo** | Only myself |
| D2 | 👥 **Internal** | Team/company internal |
| D3 | 🌍 **External** | External users/open source |

### Q2: What standard do you want?

| Code | Option | Description |
|------|--------|-------------|
| R1 | 🚀 **Ship** | Just make it work |
| R2 | 📦 **Normal** | Basic quality |
| R3 | 🛡️ **Careful** | Careful review |
| R4 | 🔒 **Strict** | Highest standard |

### Q3: How critical? (Conditional)

> **Only ask if:** (D2 or D3) AND (R3 or R4)

| Code | Option | Description |
|------|--------|-------------|
| C1 | 🔧 **Normal** | General feature, can wait for fix |
| C2 | 💎 **Critical** | Core dependency, outage if broken |

### Quick Lookup Table

| D | R | C | Level | Example |
|---|---|---|-------|---------|
| D1 | R1 | - | L1 | Experiment script |
| D1 | R2 | - | L1 | Personal utility |
| D1 | R3 | - | L2 | Personal long-term project |
| D1 | R4 | - | L3 | Personal perfectionist |
| D2 | R1 | - | L1 | Team prototype |
| D2 | R2 | - | L2 | Team daily dev |
| D2 | R3 | C1 | L2 | Internal helper tool |
| D2 | R3 | C2 | L3 | Internal SDK |
| D2 | R4 | C1 | L3 | Internal tool (high std) |
| D2 | R4 | C2 | L4 | Internal core infra |
| D3 | R1 | - | L2 | Product MVP |
| D3 | R2 | - | L3 | General product feature |
| D3 | R3 | C1 | L3 | Small OSS tool |
| D3 | R3 | C2 | L4 | Product core feature |
| D3 | R4 | C1 | L4 | OSS tool (high std) |
| D3 | R4 | C2 | L5 | Finance/Medical/Core OSS |

**For detailed explanations:** See [positioning.md](references/positioning.md)

> **Fallback:** If the user skips positioning or says "just review it," default to **L3 (Team)** and note in the report header: `**Project Positioning:** L3 Team (default — user skipped calibration)`.

---

## Level Definitions

| Level | Name | Key Question |
|-------|------|--------------|
| **L1** | 🧪 Lab | Does it run? |
| **L2** | 🛠️ Tool | Can I understand it next month? |
| **L3** | 🤝 Team | Can teammates take over? |
| **L4** | 🚀 Infra | Will others suffer if I break it? |
| **L5** | 🏛️ Critical | Can it pass audit? |

---

## Review Workflow

Follow this sequence for every review:

1. **Calibrate** — Ask Q1/Q2/Q3 → determine strictness level (or apply L3 fallback)
2. **Scope** — Confirm what to review: PR diff (changed files + immediate context), module, or specific files. If PR exceeds the level's size limit, flag it as an issue
3. **Language check** — Identify paradigm; read [language-adjustments.md](references/language-adjustments.md) if language is NOT Java/C#
4. **Review** — Walk through the 15-Point Checklist against the code
5. **Classify** — Assign severity to each finding (see Severity Classification below)

> **Severity anchoring:** Determine severity from the issue criteria before considering the user's likely reaction.

6. **Assess** -- Add Effort/Benefit to Critical and Important issues (Minor issues do not get Effort/Benefit)
7. **Report** — Generate report using the template
8. **Verdict** — Apply verdict criteria to reach conclusion

> **Review type adjustments:** For bug fixes, emphasize correctness and regression tests. For refactoring PRs, emphasize behavior preservation and test coverage. For new features, emphasize design and architecture. For test code, relax DRY tolerance.

> **Priority order:** security > correctness > design > style. Rules serve the code, not vice versa.

> **Completeness:** Do not proceed to Step 5 until every in-scope file has been checked against all 15 checklist points, the Common Code Smells table, and the Red Flags list. Mark points as N/A where legitimately inapplicable, but do not skip them.

---

## Strictness Matrix & Metric Thresholds

**Quick reference:**
- Function length: L2(≤80) → L3(≤50) → L4(≤30) → L5(≤20)
- Parameter count: L2(≤7) → L3(≤5) → L4(≤3) → L5(≤2)
- Test coverage: L2(30%) → L3(60%) → L4(80%) → L5(95%)

**For complete matrices:** See [positioning.md](references/positioning.md#strictness-matrix)

### ⚠️ Measurement Rules (MUST follow)

1. **Count logic lines only** — exclude docstrings, comments, blank lines
2. **Metrics are conversation starters, not hard gates**
3. **Do NOT report as issues (function length):**
   - Single-responsibility functions that cannot be meaningfully decomposed
   - Pure data builders, large switch/match statements, configuration mappings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zhen-Bo/pragmatic-clean-code-reviewer](https://github.com/Zhen-Bo/pragmatic-clean-code-reviewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
