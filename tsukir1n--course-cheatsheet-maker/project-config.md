---
trigger: always_on
description: Use this skill when the user wants Codex to analyze course knowledge sources and question sources organized as materials/knowledge and materials/questions, infer assessment source types, estimate exam topic importance, and generate compact A4 landscape printable cheatsheets in HTML, with optional manual PDF export from the browser. Trigger for exam review, midterm/final review, one-page cheatsheet, formula sheet, bilingual Chinese-English review sheet, topic importance ranking, Safe Review Mode,
---


# course-cheatsheet-maker

Use this skill to help Codex analyze course materials, rank topic importance from evidence, and produce compact A4 landscape cheatsheets. Codex/the agent does the reasoning; scripts only extract source text, render already prepared content, or validate workflow contracts.

## When To Use

Use for exam review, midterm/final review, one-page cheatsheets, formula sheets, bilingual Chinese-English review sheets, topic maps, and topic importance ranking.

Do not use for unrelated summarization, generic study notes, or claims about what will definitely appear on an exam.

## Inputs

Use only this simplified material structure for real courses:

```text
courses/COURSE_NAME/
  materials/
    knowledge/
    questions/
  working/
  outputs/
```

Do not ask the user to sort materials into many detailed folders. The user only needs `knowledge/` and `questions/`. Subfolders inside those two folders are allowed.

## Parameter Model

Use five separate parameters:

- `workflow_mode`: `safe-review` or `full-auto`. This controls automation only. It is selected by the user request or CLI `--mode`; do not treat Full Auto as a global content default.
- `layout`: `3col` or `4col`. This selects the preferred rendered layout for validation and review. Full Auto still renders both HTML files.
- `target_pages`: integer bounded content/page budget. User-facing "4+" means `target_pages >= 4`; scripts and `run_config.md` must record an integer.
- `coverage_mode`: `exam-compact`, `balanced-standard`, or `comprehensive-review`. This controls which topics are included.
- `detail_mode`: `simple`, `balanced`, or `detailed`. This controls how much detail included topics receive.

Resolve missing parameters before extraction and write `working/run_config.md` first so the run goal is established before material processing.

Defaults:

```text
workflow_mode = safe-review
layout = 3col
target_pages = 1
coverage_mode = balanced-standard
detail_mode = balanced
```

Special comprehensive default: if the user asks for "comprehensive", "full review", "cover all slides", "全面复习", or similar without specifying page count, use `coverage_mode: comprehensive-review` and `target_pages: 2`. Keep `detail_mode: balanced` unless the user explicitly asks for detailed output.

Natural language inference examples:

- "one-page", "一页" -> `target_pages = 1`
- "two-page", "两页" -> `target_pages = 2`
- "three-page", "三页" -> `target_pages = 3`
- "final exam", "midterm", "考试速记", "只要重点" -> `coverage_mode = exam-compact`
- "standard review", "正常复习版", "标准版" -> `coverage_mode = balanced-standard`
- "comprehensive", "全面复习", "cover all slides" -> `coverage_mode = comprehensive-review`
- "simple", "concise", "quick reference", "简洁", "速查" -> `detail_mode = simple`
- "detailed", "explain more", "学习版", "详细版" -> `detail_mode = detailed`

Resolve conflicts in this priority order:

1. `target_pages`
2. `layout`
3. `coverage_mode`
4. `detail_mode`
5. `workflow_mode`

`target_pages` controls total content capacity, not an exact printed-page guarantee and not the per-topic writing depth. For example, if the user asks for a "one-page detailed comprehensive review", keep `target_pages = 1` and preserve requested detail for A topics first while compressing lower-priority content.

Coverage modes:

- `exam-compact`: A topics must be included; B topics should fit if space allows; C topics only if space allows; R topics omitted. Past-paper evidence has the strongest influence. Lecture-only topics may be compressed or omitted unless foundational.
- `balanced-standard`: default content coverage. A topics must be included; B topics should be included; C topics should be representatively included; R topics omitted. Lecture coverage decides minimum inclusion, while exam evidence strongly increases priority and usually justifies expanded treatment.
- `comprehensive-review`: not constrained to one A4 page. A and B topics must be included; C topics should be included if useful; R topics are generally omitted unless explicitly useful as appendix/reference material.

Representative C-topic rule: for `coverage_mode = comprehensive-review` and `target_pages >= 2`, include representative C topics when meaningful C candidates exist. Do not reduce C topics to zero unless there are no meaningful C candidates. Compress repeated examples, repeated traps, and Q/A before removing all representative C topics. For a 2-page `3col` comprehensive review, aim for 6-12 representative C topics where useful.

Target page budget policy for 3col:

- `target_pages = 1`: floor 14k chars, target 16k-18k, soft ceiling 20k, hard ceiling 22k; topic target 35-45; A all compact, B selected high-value, C 0-3 representative, R omit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsukiR1n/course-cheatsheet-maker](https://github.com/tsukiR1n/course-cheatsheet-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
