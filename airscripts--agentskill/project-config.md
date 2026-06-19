---
trigger: always_on
description: Let any agent produce code indistinguishable from the existing codebase.
---


# SKILL.md — agentskill

> **Operational spec for agentskill.**
> This file governs _when_ to invoke, _what_ to run, and _in what order_.
> For _how_ to generate `AGENTS.md`, read [`SYSTEM.md`](./SYSTEM.md) — it is the behavioral bible.
> These two files are complementary. Neither is sufficient alone.

---

## Purpose

Analyze one or more code repositories. Extract exact coding conventions. Synthesize a precise, forensic `AGENTS.md` that allows any agent to produce code indistinguishable from the existing codebase.

---

## Generation Modes

agentskill supports two generation modes. The mode determines who authors the
final document:

### AI-led generation (skill mode — this file)

The model synthesizes the final `AGENTS.md` itself. CLI analyzer commands are
used **only for evidence gathering** — to extract repository facts that the
model cannot derive reliably from reading source files alone.

**In skill mode, never call `agentskill generate` to produce the final
`AGENTS.md`.** The model is the author. Analyzer output is the raw material,
not the finished product.

### CLI static generation (operator mode)

The user runs `agentskill generate` directly. The packaged runtime emits
markdown automatically. This is appropriate for deterministic direct generation
without an LLM in the loop.

**Use CLI generation only in non-LLM static/operator workflows where the user
explicitly wants tool-generated markdown rather than AI-authored synthesis.**

---

## Rule: AI Authorship

> **The model authors the final document in skill mode.**

- Do not use `agentskill generate` or `python scripts/generate.py` to produce
  the final `AGENTS.md` when operating as a skill or in any AI-assisted
  workflow.
- Use analyzer commands (`analyze`, `scan`, `measure`, `config`, `git`, `graph`,
  `symbols`, `tests`) to gather repository facts.
- The final generated markdown must be synthesized by the AI from analyzer
  evidence, direct source file reads, and supporting documentation.
- Treat analyzer outputs as evidence, not as the final authored document.

---

## Trigger Phrases

Invoke this skill when the user says any of the following — or a close paraphrase:

- _"Generate an AGENTS.md"_
- _"Extract my coding style"_
- _"Analyze my repo for conventions"_
- _"Create a style guide from my code"_
- _"Update my AGENTS.md"_
- _"My agent doesn't write code the way I do — fix it"_

Do **not** invoke this skill for general code review, refactoring, or style advice not tied to generating `AGENTS.md`.

---

## File Ecosystem

| File                     | Role                                                                                   |
| ------------------------ | -------------------------------------------------------------------------------------- |
| `SKILL.md` _(this file)_ | Operational spec: workflow, scripts, fallbacks, uncertainty handling                   |
| `SYSTEM.md`              | Behavioral spec: what to generate, section by section, and how to evaluate it          |
| `references/GOTCHAS.md`  | Extraction errors to avoid; update this file whenever a new failure mode is discovered |
| `examples/`              | Analyzer fixtures plus reference `AGENTS.md` examples; consult when handling an unfamiliar repo shape |

> **Maintenance rule:** If SYSTEM.md and SKILL.md ever contradict each other, SYSTEM.md wins. Fix SKILL.md to match.

> **Availability rule:** If this skill was downloaded from ClawHub, or if `examples/` is unavailable locally, do not consult `examples/`; skip it to avoid execution errors.

---

## Workflow

Execute these steps **in order**. Do not skip steps. Do not reorder steps.

---

### Step 1 — Collect

Ask the user for repo path(s). Accept one or more. Confirm before proceeding.

```
Provide the path(s) to your repository or repositories.
One path per repo. Multiple repos are supported.
```

If the user provides a monorepo, note this explicitly — steps 3 and 4 of SYSTEM.md apply.

---

### Step 2 — Scan

Run the scan script to get the directory tree and source file inventory.

```bash
python scripts/scan.py <repo>
```

**Outputs:** annotated directory tree, source files grouped by language with line counts.

**Use the output to decide what to read** — largest files first, entry points and core modules before tests.

> **If the script fails:** Manually walk the directory tree using available file tools. Note in your working context that the scan was manual — this affects reliability of the file inventory for large repos.

---

### Step 3 — Measure

Run the measurement script to get exact formatting metrics.

```bash
python scripts/measure.py <repo>
python scripts/measure.py <repo> --lang python   # single language
```

**Outputs:** per-language indentation unit and size, line length percentiles (p95 and p99), blank line distributions between top-level definitions and between methods, trailing newline convention.

> **If the script fails:** Proceed without exact measurements. Mark all formatting measurements in the generated `AGENTS.md` as `[tentative]` and note that manual inspection was used. Do not estimate percentiles — state the observable range instead.

---

### Step 4 — Config

Run the config script to detect formatters, linters, and their exact settings.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airscripts/agentskill](https://github.com/airscripts/agentskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
