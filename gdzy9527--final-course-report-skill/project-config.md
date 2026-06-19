---
trigger: always_on
description: [English](AGENTS.md) | [中文](AGENTS.zh-CN.md)
---

# Final Course Report Agent Guide

[English](AGENTS.md) | [中文](AGENTS.zh-CN.md)

Use this guide when the user asks for a final course report, course summary report, report with a supporting practice project, Word/PDF report generation, report figures, project screenshots, or a reusable report-generation workflow.

## Operating Principles

- Build a real, runnable, reproducible project before writing the project-practice chapter.
- Except for purely hardware-only courses, every new software or software-hardware project must include a complete frontend and backend or equivalent software stack.
- Treat older NLP examples without a frontend as historical limitations only. Do not copy that pattern into new work.
- Never invent experiment results, screenshots, code snippets, metrics, build success, frontend behavior, or API availability.
- If network, API access, model downloads, browser screenshots, compilers, or dependencies are unavailable, provide a runnable offline fallback when possible, update requirements and environment notes, and record the limitation in the report and delivery notes.
- The user creates conda environments and installs dependencies manually. Write `requirements.txt` and tell the user what to install; do not assume dependencies are already present.
- Ask for cover-page fields before final report generation: name, student ID, class, teacher, course, template path, and conda Python path. Treat project direction and target course directory as optional overrides: infer a suitable project from the course when omitted, and propose/create a course directory from the course name under the current workspace or user-approved base directory when omitted.
- Prefer useful inference over asking. Ask only for facts that are required, cannot be discovered locally, and would be risky to assume.
- Default to UTF-8 for Chinese Markdown, JSON, CSV, and text files.

## Required Workflow

1. Inspect the target directory, template document, existing project files, and user-provided constraints.
2. Create or update a deliverable-grade project with clear modules, frontend, backend/core logic, data, README, requirements, run scripts, tests, and results.
3. Run or document the exact verification commands using `{conda_python}`.
4. Generate deterministic project assets: directory tree, architecture/system flow, code screenshots, frontend screenshots, metric charts, logs, and result tables.
5. Write the report from the template and real project outputs.
6. Include references/sources by default.
7. Render and visually inspect the report when document tools are available.
8. Prepare a delivery package after showing the proposed exclusion list for confirmation.

## Reference Files

Read only the files needed for the task:

- `references/report_workflow.md` for the report process and chapter structure.
- `references/project_contract.md` for required project outputs and delivery-grade behavior.
- `references/frontend_quality.md` for mandatory frontend requirements.
- `references/visual_rules.md` for academic-style figure requirements.
- `references/environment.md` for conda and dependency handling.
- `references/delivery_quality.md` for quality gates and packaging.
- `references/originality_and_sources.md` for personal writing, anti-generic prose, and references.
- `references/course_patterns.md` for lessons from the existing C++, LLM app, and NLP reports.
- `references/smart_defaults.md` for minimal prompts, course-type heuristics, and automatic project/directory selection.
- `references/portability.md` for cross-machine and cross-agent conventions.

## Output Expectations

Final responses should list:

- report path
- project path
- environment path used
- generated figures and tables
- verification commands and results
- known limitations
- delivery package contents or pending packaging confirmation

## Hardening From Cross-Agent Tests

- Do not deliver thin reports: plan density first and verify with `references/report_density.md`.
- Do not reuse generic templates: write a course-specific project brief before coding.
- Figures must be light-background, complex enough for academic reports, printable, and embedded in the DOCX.
- Clean old template body content and remove stale course keywords.
- Organize appendices by purpose instead of dumping unrelated fragments.
- Maintain `results/execution_log.md` for long runs, including commands, outputs, failures, and fallbacks.
- Run available audit scripts before delivery; fix failures or record accepted limitations honestly.

## Hard Rules After 2.0 Regression Tests

- Every report must use a new project; do not reuse examples, prior test projects, or old same-course projects.
- Before coding, create `results/project_brief.md` with three candidate projects and anti-reuse reasoning.
- Before frontend coding, create `results/frontend_design_brief.md`; the final UI must reflect the project workflow and must not be the same blue dashboard every time.
- If student name, ID, class, or teacher are missing, do not ask at the start; leave cover fields for the user to fill in Word.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GDZY9527/final-course-report-skill](https://github.com/GDZY9527/final-course-report-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
