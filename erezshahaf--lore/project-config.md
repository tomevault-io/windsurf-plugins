---
trigger: always_on
description: description: How to improve Lore from eval failures using pipeline traces—no test-cheating, no overfitting.
---

---
description: How to improve Lore from eval failures using pipeline traces—no test-cheating, no overfitting.
alwaysApply: false
globs:
  - evals/**
  - skills/skill-classification/**
  - electron/services/**
  - .cursor/rules/**
---

# Agent improving — evals, traces, and honest fixes

Use this when you are asked to diagnose eval failures, tune the agent, or change prompts and skills based on scenario results. Use it as the **policy** for: inspect artifacts → trace-backed fixes. **Do not** re-run promptfoo or other eval suites from the agent unless the user **explicitly** asks you to; verification is the human’s or CI’s step.

Fragment shape for skills lives in [`prompt-modules.mdc`](prompt-modules.mdc); this file is process and discipline for eval-driven changes.

## Purpose

- Ground fixes in **what actually happened** in the pipeline (classifier output, retrieval, composed reply, and so on), not only in the final pass or fail string.
- Prefer changes that **generalize**: they should help whole *classes* of user inputs, not one scenario’s exact wording.
- **Never pass a test by breaking product behavior** that the test does not cover.

## Inputs (each iteration)

1. Prefer the **newest full run JSON** under `evals/results/`: files named like `promptfoo-<suite>-<timestamp>.json`. That file has complete `pipelineTrace` data. When present, [`evals/results/.promptfoo-latest.json`](evals/results/.promptfoo-latest.json) points at the raw result file the last summary used.
2. Use `*-summary.json` or `*-summary.md` only for a quick overview; they may truncate or sample traces.
3. For each failure, record **scenario id**, **step**, **failed checks or judge output**, and align with the **scenario rubric** so you know *which stage* should have behaved differently.
4. When the artifact makes it clear, tag the failure as a **deterministic check** (counts, schema, structured assertions) vs an **LLM judge** verdict. Prioritize trace-backed fixes for deterministic failures. Repeated judge disagreements on similar outputs may indicate rubric ambiguity or judge-prompt issues rather than product prompts alone.

## Diagnosis workflow (do not skip)

1. **Enumerate** failing scenarios from the full JSON.
2. For each failure, open **`pipelineTrace`** for the relevant assistant turn (and earlier turns if the scenario is multi-step).
3. Name the **failing stage** (for example classification, retrieval, worker prompt, reply composition).
4. Write a **one-line hypothesis** (for example “classifier merged two intents”, “wrong `decisions` branch”, “composer dropped structured facts”).
5. Prefer **one coherent change set per iteration** (not a grab bag): fix the **earliest wrong stage** first, then **re-read the trace**. Touch downstream prompts or handlers only if the failure persists after upstream is correct.
6. Keep the change set **small** for that iteration—address the hypothesis class without unrelated edits.

## Closing the iteration (mandatory)

1. After edits, **do not** start eval runs (for example `scripts/run-promptfoo.mjs`) on your own. Tell the user what changed and suggest **they** (or CI) run the relevant suite when they want a fresh pass/fail signal.
2. If the user pastes new results later, compare **pass or fail delta** (counts or scenario ids) and call out **new** failures. Fixing one scenario while regressing others is a sign of overfitting; widen or revert the change.

## Comparing eval runs

- Compare **the same suite** and, when possible, the **same repeat count** (`--repeat`); otherwise interpret pass rates carefully.
- **Row-level** pass % can mislead when total rows differ (for example 46 trials vs 138). Prefer **per-scenario** pass counts or the **mean of per-scenario pass percentages** from `*-summary.json` when both runs list the same scenario ids.
- If **aggregate** pass/fail counts are unchanged but **which scenarios** pass or fail **swaps** between runs, treat that as **variance** (and repeat/temperature effects) before blaming a specific prompt tweak.

## Prompt budget (300 tokens per contributing fragment)

Skills with `decisions/` trees merge multiple `entry.md` files via `assembleForkedSkillPrompt` in [`electron/services/skillLoader.ts`](electron/services/skillLoader.ts). The budget applies **per file**: each **`entry.md` body** you add or edit under `skills/skill-classification/` (after trim) should stay within **300 tokens** of instructional content.

- **Why per file**: Assembly joins many fragments; keeping each `entry.md` small forces **narrow leaves** and clear routing instead of one overloaded blob.
- **How to estimate**: Prefer a real tokenizer when available. If you only need a quick check, **approximate** with `ceil(characterCount / 4)` on the trimmed UTF-8 body (English-heavy markdown). Values near the ceiling should be tightened or split.
- **If content does not fit**: **Split**—add or deepen `decisions/<decisionKey>/<value>/` branches, move cross-cutting lines into a parent that only routes, and keep detailed rules in leaves. Do not “vague out” constraints just to save tokens.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ErezShahaf/Lore](https://github.com/ErezShahaf/Lore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
