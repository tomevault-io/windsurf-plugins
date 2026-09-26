---
trigger: always_on
description: You are working in the **spreadsheetbench** env of SkillOpt. The workspace
---

# SkillOpt spreadsheetbench — Copilot context

You are working in the **spreadsheetbench** env of SkillOpt. The workspace
root opened in this VS Code window is `copilot_example/spreadsheetbench/`.

## Model-specific workspaces

Workspaces are **isolated by target model** to prevent runs from clobbering
each other. When you run `bash run.sh --target_model <M>`, the script
auto-creates `workspaces/<model>/` (e.g. `workspaces/gpt-5.5/`,
`workspaces/gpt-5.4-nano/`). Each workspace has its own `skill.md`,
`.skillopt/samples/`, and `.skillopt/history/`.

For backward compatibility, `run.sh` also maintains a `workspace` symlink
pointing to the **most-recently-used** model's workspace, so existing
prompts and commands that reference `workspace/skill.md` keep working
against whichever model was last run.

## Files you care about

- `workspaces/<model>/skill.md` — **the skill being optimized** for that model
- `workspaces/<model>/.skillopt/samples/failed/*.md` — failed tasks
- `workspaces/<model>/.skillopt/samples/passed/*.md` — successful tasks
- `workspaces/<model>/.skillopt/history/` — snapshots of `skill.md` before edits
- `workspaces/<model>/.skillopt/_eval_run/<ts>/results.jsonl` — eval output
- `workspaces/<model>/.skillopt/_eval_run/<ts>/predictions/<task_id>/conversation.json` —
  full ReAct trajectory
- `workspaces/<model>/.skillopt/_eval_run/<ts>/predictions/<task_id>/solution.py` —
  the agent's last generated script (for diagnosing exec failures)
- `skills/initial.md` — **shared baseline skill** (copied to each model's
  `workspace/skill.md` on first run; read-only reference thereafter)
- `run.sh` — runs eval; auto-selects workspace based on `--target_model`

## Task summary

- **Spreadsheet code-generation**: each task gives a natural-language
  instruction (e.g. *"In sheet 'Q3 Sales', sort rows by column C in
  descending order and put the top 5 in cells F2:J6"*) plus an input
  `.xlsx` file. The agent runs in a **ReAct** loop: reads/inspects the
  workbook, writes a `solution.py` (using `openpyxl` + optionally
  `pandas`), executes it, and produces an output `.xlsx`.
- A task usually has **multiple test cases** (different input files
  exercising the same instruction). Each case's output is compared
  against the gold via cell-level / sheet-level matching.
- **Target model**: `gpt-5.5`, reasoning `medium`. Code-gen + reasoning
  benefits from the bigger model. The `select_endpoint_for_model` helper
  in `../env.sh` lets you swap to `gpt-5.4`, `gpt-5.4-mini`,
  `gpt-5.4-nano`, or `qwen3.5-9b` without touching this folder. See
  **Switching target models** below.
- **Per-task budget**: up to `max_turns=30` ReAct turns,
  `max_completion_tokens=16384`, **`exec_timeout=600s`** for
  `python solution.py`.
- **Scoring** (`eval_only.py`):
  - `hard` = **all test cases pass for this task** (1 or 0).
  - `soft` = `n_pass / n_cases` (fraction of cases pass — partial credit).
  - Both averaged across tasks. `hard` is what `/skillopt-loop`'s gate
    decides on; `soft` often moves first when the agent is *almost* there.

## Sample file format

Each `.md` under `samples/{failed,passed}/`:

```markdown
---
id: <task_id>
status: failed | passed
score: 0.0 | 1.0
timestamp: <iso>
env: spreadsheetbench
tags: [cell_level | sheet_level | other, <fail_reason snippet>]
---
## Input
(natural-language instruction)
_meta_: task_type=<...>, instruction_type=<...>, n_cases=<...>
## Expected
(usually empty — gold lives in answer xlsx files, not in results.jsonl)
## Agent output
(usually empty — outputs are .xlsx files, not text)
## Trace
<details><summary>Full trajectory</summary>
the conversation.json contents (ReAct turns)
</details>
## Notes
fail_reason: agent-error | exec-failed | wrong-cells | partial-cases | timeout | ...
```

The trace is loaded from
`_eval_run/<ts>/predictions/<task_id>/conversation.json` and truncated at
8000 chars. Use `## Notes` (`fail_reason`) and the **last 3–6 ReAct turns**
to triage; expand only if you need to see what the agent originally
inspected.

## Editing principles for `workspace/skill.md`

1. **Library-selection guidance is the highest leverage knob.** Many
   failures come from `pandas.to_excel()` clobbering formulas / merged
   cells / formatting. Sharpening "when to use openpyxl vs pandas" (and
   the warnings about `to_excel`) usually moves several tasks at once.
2. **Cluster by `task_type`.** Failures fall into:
   - `cell_level` — write specific cells / ranges (often A1-style refs).
   - `sheet_level` — produce / modify entire sheets, headers, sorts,
     filters, summaries.
   - `other` — usually formula-heavy or cross-sheet.
   Patch the dominant cluster first; cell_level fixes rarely help
   sheet_level and vice versa.
3. **Concrete > abstract.** Replace prose like *"preserve formatting"*
   with a code snippet showing `wb = openpyxl.load_workbook(...);
   wb.save(...)`. The model imitates examples in `skill.md` more
   reliably than it follows declarative rules.
4. **Cite samples by id** (e.g. `samples/failed/<task_id>.md`) plus the
   ReAct turn number where the failure pattern appears, so the diff is
   auditable.
5. **Snapshot before edit.** Always
   `cp workspace/skill.md workspace/.skillopt/history/<ts>__before.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvolvingLMMs-Lab/SkillOpt-Lite](https://github.com/EvolvingLMMs-Lab/SkillOpt-Lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
