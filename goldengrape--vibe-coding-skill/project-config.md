---
trigger: always_on
description: Help beginners and non-programmers do safer vibe coding by turning a rough software idea into a small, clear, AI-readable project plan before implementation. Use for new project ideas, MVP planning, URD/ADD/MDD/TDD/RMD docs, Python uv setup, git checkpoints, PR/merge safety, trace updates, and OKF AI notes.
---


# Vibe Coding Skill

Use this skill when a user has a new software idea and wants to use an AI coding assistant to build it from scratch without drifting, overbuilding, or losing track of decisions.

The public promise is simple:

```text
Turn a rough idea into a clear, testable project map for safer vibe coding.
```

The user does not need to know SDD, URD, ADD, MDD, TDD, RMD, axiomatic design, or coupling analysis. Those terms are internal machinery. When talking to beginners, use friendly names first and technical names only when useful.

## Beginner-facing names

Use these names in normal conversation:

| Friendly name | Internal file | Meaning |
| --- | --- | --- |
| Idea Brief | `docs/URD.md` | What the user wants, who it is for, what counts as success. |
| Design Split | `docs/ADD.md` | How the idea is split into independent parts before coding. |
| Building Blocks | `docs/MDD.md` | The modules, interfaces, data, and contracts. |
| Check Plan | `docs/TDD.md` | How we know the project works. |
| Build Path | `docs/RMD.md` | The safest order for the AI to implement things. |
| Project Map | `docs/TRACE.md` | Links from idea → design → code tasks → tests. |
| Project Setup | `pyproject.toml`, `.gitignore`, source and test folders | The starter files needed before coding. |
| Git Checkpoint | git branch, commit, pull request, merge | A safe save-and-review point after each build slice. |
| AI Notes | `okf/` | Short pages that help an AI retrieve the right context. |

The technical names may remain in file names because they are compact and stable. Do not force the user to learn the acronyms before they can use the skill.

## Directory layout

```text
pyproject.toml       # for Python projects, managed with uv
.gitignore
README.md
src/ or app/
tests/

docs/
  URD.md   # Idea Brief / User Requirement Document
  ADD.md   # Design Split / Axiomatic Design Document
  MDD.md   # Building Blocks / Module Design Document
  TDD.md   # Check Plan / Test-Driven Document
  RMD.md   # Build Path / Route/Runbook/Execution Map Document
  TRACE.md
  CHANGELOG.md
  PARKING_LOT.md

okf/
  index.md
  log.md
  terms/
  requirements/
  decisions/
  modules/
  interfaces/
  tests/
  paths/
  issues/
  references/

.vibe/
  trace.json
  doc_state.json
  coupling_history.json
  update_log.json
```

`docs/` is the source of truth. `okf/` is a derived AI-readable knowledge layer. `.vibe/` contains machine-readable state and trace files.

This skill is optimized for new projects. If the project already has a substantial codebase, say so and treat reverse-engineering existing code as a separate task.

## Core behavior

### 0. Operating guardrails

Use these guardrails in every mode. They prevent the AI from moving from planning to coding too early.

#### 🔴 CHECKPOINT moments

Pause and show the user the current decision before continuing when any of these happen:

| Moment | What to show | Continue only when |
| --- | --- | --- |
| Idea Brief is complete | target user, core task, scope, non-scope, constraints, assumptions, open questions | the user accepts or gives corrections |
| Design Split is classified | FR/DP list, design matrix, classification, retry log | uncoupled/decoupled design is accepted, or accepted coupling is explicitly recorded |
| Build Path is ready | first 3 RMD tasks, test commands, rollback points, Git checkpoint plan | the user agrees this is the implementation order |
| First real push or merge | remote name, branch, PR/merge action, tests run | the user or repo rules approve the remote action |
| Document rewrite would overwrite existing work | files to change, backup or diff summary | the user confirms overwrite or chooses a smaller edit |

#### 🛑 STOP conditions

Stop the current action and route the issue back to the right document when any condition below is true:

| Trigger | Required action |
| --- | --- |
| The user asks to code before URD has a user, task, scope, and success criterion | explain the missing decision and ask one concrete question, or create a labeled assumption if it does not block the next step |
| ADD remains coupled after 3 structural retries | create an Accepted Coupling record with risks and guard tests before implementation |
| A test has no oracle | update TDD before writing implementation code |
| MDD lacks an interface contract for a public API | update MDD before writing that API |
| OKF contradicts docs | write `okf/issues/PROB-xxxx.md`, then fix `docs/` first |
| Git working tree contains unrelated changes | show `git status`, avoid committing unrelated files, and ask before touching them |
| A push, merge, deletion, secret exposure, or irreversible operation is needed | ask for explicit permission before executing |

#### Failure handling table

| Failure mode | First response | Fallback if still blocked |
| --- | --- | --- |
| User gives a vague idea | ask one high-value question with 2–3 options | record assumptions and proceed with simple docs only |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goldengrape/vibe-coding-skill](https://github.com/goldengrape/vibe-coding-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
