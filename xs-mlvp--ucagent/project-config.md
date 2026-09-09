---
trigger: always_on
description: This file contains repository-level instructions for coding agents working on
---


# AGENTS.md

This file contains repository-level instructions for coding agents working on
UCAgent. It applies to the entire repository unless a more specific AGENTS.md is
added below a subdirectory.

## Project Goal

UCAgent is a Python 3.11+ AI agent for hardware verification. It combines:

- configuration-driven verification workflows;
- stages and stage lifecycle management;
- deterministic checkers that gate stage completion;
- local and MCP-exposed tools;
- runtime Guide_Doc files, output templates, and optional skills;
- UnityChip/toffee tests, coverage, Bug analysis, and waveform evidence.

The main engineering objective is not merely to make an LLM produce output. The
system must prevent incomplete DUT specifications, invalid verification results,
and fabricated evidence from being accepted.

## Repository Map

- `ucagent/verify_agent.py`: top-level agent assembly and runtime initialization.
- `ucagent/cli.py`, `ucagent.py`: installed and source-tree CLI entry points.
- `ucagent/setting.yaml`: global defaults, backends, tools, launch settings, and
  environment-backed defaults.
- `ucagent/util/config.py`: layered config loading, overrides, template handling,
  and the shared `.ucagent/runtime_config.json` snapshot.
- `ucagent/stage/`: `VerifyStage`, `StageManager`, Check/Complete lifecycle, stage
  history, and stage tools.
- `ucagent/checkers/`: deterministic validation and batch-task implementations.
- `ucagent/tools/`: LLM tools, file/test operations, MCP conversion, skills, and
  waveform analysis.
- `ucagent/abackend/`: LangChain and command-line backend adapters.
- `ucagent/server/`, `ucagent/tui/`: master APIs, terminals, web UI, and TUI.
- `ucagent/lang/zh/config/default.yaml`: the main UnityTest workflow, system
  prompts, stage tasks, checker wiring, and stage-local skills.
- `ucagent/lang/zh/doc/Guide_Doc/`: runtime guidance copied into a DUT workspace.
- `ucagent/lang/zh/template/`: files rendered into a DUT workspace.
- `ucagent/lang/zh/skills/`: skills copied to `<workspace>/.ucagent/skills` when
  skill support is enabled.
- `docs/content/`: developer/user documentation for the MkDocs site. This is not
  the same as runtime `Guide_Doc`.
- `tests/`: unit and focused regression tests.
- `examples/`: example DUT inputs and, in some cases, generated verification
  artifacts. Do not treat generated example outputs as canonical source.

## Working Tree Discipline

- Assume the worktree may already contain important user changes. Inspect
  `git status --short` and the relevant diff before editing.
- Never revert, normalize, or reformat unrelated changes. Some files use CRLF and
  some use LF; preserve the existing file's line endings.
- Keep patches scoped by feature. Do not combine prompt rewrites, checker changes,
  server refactors, and unrelated cleanup unless the task requires all of them.
- Do not edit generated `output/`, DUT workspaces, waveform files, `.ucagent`
  checkpoints, or stage-history repositories unless the task explicitly targets
  generated/runtime state.
- Do not run broad destructive Make targets such as `make clean`, `reset_%`, or
  `clean_%` in a shared worktree without explicit authorization. They remove
  outputs, waveforms, checkpoints, and generated workspaces.
- Do not create a commit unless explicitly requested. When requested, group
  commits by coherent feature and use clear English commit messages.

## Documentation Scope Contract

Documentation updates must follow the subject and abstraction level of each
document. A runtime behavior change does not authorize repeating its details in
every document that happens to mention the affected feature.

- Put feature-specific behavior, limitations, architecture, backend selection,
  command availability, and operational caveats in that feature's dedicated
  documentation. Update other documents only when their own instructions would
  otherwise be incorrect or incomplete for the task they teach.
- Keep `README.md`, `README.zh.md`, and `README.en.md` focused on their existing
  high-level introduction and onboarding flow. Do not add implementation details
  or special-mode explanations merely to advertise or restate a code change.
- Keep general CLI option references concise: document the public option syntax
  and its direct purpose. Put mode-specific backend semantics, lifecycle details,
  and command restrictions in the mode's dedicated documentation instead of
  expanding the generic option table.
- Do not rewrite existing setup steps, environment-variable examples, or command
  examples in unrelated documents unless the change is necessary for that
  document's own workflow or the task explicitly requests it.
- Prefer one authoritative explanation and link to it when another document
  genuinely needs more detail. Do not duplicate the same explanatory paragraphs
  across README files, option references, tutorials, and feature guides.
- For Master mode specifically, document the Master process's blank backend,
  separation from child-Agent backend configuration, retained PDB control
  surface, and unsupported model-work commands only in
  `docs/content/02_usage/07_web_master.md`. Do not add those details to README
  files or the general CLI option reference.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XS-MLVP/UCAgent](https://github.com/XS-MLVP/UCAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
