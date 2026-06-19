---
trigger: always_on
description: Audits and optimizes Claude-directing artifacts (CLAUDE.md, SKILL.md, subagent, hook, MCP config, system or user prompt, workflow, api_config) against 189 cited Anthropic rules across 11 dimensions, calibrated per model (Opus 4.7, Sonnet 4.6, Opus 4.6, Haiku 4.5). Every finding ships with verbatim quote and source URL; authorial voice is preserved.
---


# Anthropic-Grade Optimizer

Audits any Claude-directing artifact against the official Anthropic doctrine,
calibrates findings by target model, and proposes surgical optimizations that
preserve authorial voice. Every finding cites a verbatim source URL; the skill
ships with cited rules only.

## Contents

- Hierarchy of authority
- Honest scope
- Quick start (and worked examples)
- The Three Laws
- Artifact types (9)
- Target-model modulation
- The 11 dimensions
- Severity → triage
- Emphasis conflict (D9, type-aware)
- Detection methods
- Operating modes
- Adaptive modes (auto-trigger)
- Output format (concise default)
- Workflow (10 steps)
- Scope discipline (positive framing)
- Edge cases
- Self-audit: Open Questions
- References (10 files)
- Assets (canonical snippets)
- Scripts (6 entry points)
- Validation (eval suite + strange-loop)

## Hierarchy of authority

Anthropic doctrine is the sole source of scoring rules. Optional interpretive
lenses inform *how* to reason about findings (what to preserve, when to defer,
how to phrase recommendations) — they stay outside the rubric. On collision,
Anthropic always wins.

## Honest scope

189 unique cited rules across 11 dimensions, each with a verbatim quote and
source URL. Read `references/rules-anthropic.yaml` § `meta.total_unique_rules`
for the canonical count — every other counter in the skill must read from
there to prevent drift. Rules with deterministic detection (regex,
code-check) are audited automatically by `scripts/pass1_mechanical.py`. Rules
with qualitative criteria (llm-judge, heuristic) are audited by following
`references/pass2-protocol.md`. Coverage gaps and known limitations live in
`references/gaps.md` and must appear in the `coverage_caveat` block of every
report.

Pass-1 deterministic coverage in v1.2: ~52 rules (~28% of 189). Pass-2
covers the remaining ~137 qualitative rules. Hybrid detection is supported via
the `requires_pass2_grade` flag on findings.

## Quick start

**Canonical entry point (default):**

```
python scripts/run.py <artifact_path> --target opus-4-7 --mode audit
```

The orchestrator chains classify → pass 1 → score → emit, and surfaces Pass 2
prompts for the LLM. This is the single right answer for ~95% of audits.

**Manual flow** is for one specific case: an operator forcing fine-grained
control over a single phase (re-running pass 1 only, scoring an external
findings file, debugging the classifier). Do not use it as a parallel path —
the orchestrator is the contract.

1. **Ingest** — receive the artifact path or content, target model
   (default `opus-4-7`), mode (default `audit`).
2. **Classify** — Run `scripts/classify_artifact.py` to detect artifact type
   and load the rule subset from `references/rubric-by-type.yaml`.
3. **Audit** — Pass 1 mechanical via `scripts/pass1_mechanical.py`. Pass 2
   qualitative reasoning following `references/pass2-protocol.md` against
   `references/rules-anthropic.yaml`.
4. **Diagnose** — Classify each finding as 🔴 must-fix, 🟡 should-fix,
   🟢 may-fix, ❓ open-question, or ⚪ preserve (authorial voice).
5. **Optimize** *(when mode = optimize or full)* — Produce surgical diffs
   per `references/pass2-protocol.md` § Diff Generation. Each diff cites
   `source_url` + `verbatim_quote`. When a rule has a canonical snippet,
   emit a verbatim patch from `assets/snippets/` rather than paraphrasing.
6. **Validate** *(when mode = full)* — Re-score post-diff; abort when a
   hard rule is introduced or voice drift exceeds 10%.
7. **Emit** — Concise report (summary + scorecard + diff). Verbose mode
   adds reasoning trail, preservation log, and open questions.

### Worked example 1 — auditing a SKILL.md for Opus 4.7

<example>
Operator request: "audit this skill for Opus 4.7"
Artifact: `~/.claude/skills/pdf-tools/SKILL.md` (240 lines)

Step 1 — Classify: type=`skill`, has_frontmatter=true, body_lines=235.
Step 2 — Load rubric: 24 D-SKILL rules + 17 D-CLAR + 10 D-STRUCT + 8 D-EXAMPLE +
         5 D-EVAL + safety. Suppress AR-CC-S09 doctrine-conflict?  No (skill body).
Step 3 — Pass 1 fires: AR-CC-S20 (lib mentioned without `pip install`), AR-CC-S22
         (3 script refs unframed), AR-CLAR-006 (7 negatives, 1 positive alt).
Step 4 — Triage: 1 🔴 (AR-CC-S20), 2 🟢 (S22, CLAR-006).
Step 5 — Optimize emits snippet patches: none (no rule with canonical snippet
         fires). Inline diff for AR-CC-S20 adds "Install required package: pip install pypdf".
Step 6 — Validate: post-diff score 92 (was 78). Voice drift 4% — under 10% gate.
Step 7 — Emit concise report.
</example>

### Worked example 2 — strange-loop self-audit

<example>
Operator request: "run the skill on its own SKILL.md"
Artifact: `anthropic-grade-optimizer/SKILL.md`

Step 3 — Pass 1 fires: AR-CC-S14 (name contains "anthropic" reserved word) and
         possibly AR-CC-S21 (TOC) and AR-CC-S22 (script framing).
Step 4 — AR-CC-S14: see § Self-audit: Open Questions below for the operator's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [l0z4n0-a1/skill-anthropic-grade-optimizer](https://github.com/l0z4n0-a1/skill-anthropic-grade-optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
