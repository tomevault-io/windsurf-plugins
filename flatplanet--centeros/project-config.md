---
trigger: always_on
description: > Fresh harness session? If this repo has not been initialized yet, read `STARTUP.md` first. If it has already been initialized, read `BOOTSTRAP.md` first.
---

# CenterOS AI Operating System Instructions

> Fresh harness session? If this repo has not been initialized yet, read `STARTUP.md` first. If it has already been initialized, read `BOOTSTRAP.md` first.

This directory is the root of an AI operating system: a structured collection of folders that house workflows, wikis, templates, memory, and other modular components [AI_NAME] can operate on.

## CLAUDE.md / AGENTS.md Synchronization

`CLAUDE.md` and `AGENTS.md` are synchronized copies of the same instruction file and must remain identical at all times.

If you add, remove, edit, or reorganize content in either file, make the identical change to the other file during the same operation.

Never update one without updating the other.

Before finishing any operation involving either file, compare them and confirm that their contents are identical.

## Public Template Versus Personalized Instance

This repo starts as the public CenterOS template. After `STARTUP.md` runs once, it becomes a personalized AI operating system.

- Use `CenterOS` when referring to the public template/framework.
- Use `CenterOS` when referring to the user's personalized operating system.
- Use `[AI_NAME]`, `[PRINCIPAL]`, and `[PRINCIPAL_NICKNAME]` for user-specific identity values until startup replaces them.

## Directory Structure

### `workflows/`

All workflows live here. A workflow is a self-contained, runnable procedure: steps, prompts, scripts, or skills that accomplish a defined task. When creating a new workflow:

- Place it in its own subdirectory under `workflows/<workflow-name>/`.
- Every workflow directory must contain a `CONTEXT.md` file.
- Every active workflow directory must contain a `LOG.md` file.
- Keep each workflow self-contained. Inputs, outputs, dependencies, and steps should be documented inside its folder.

### Workflow-Scoped Skills

A skill is a directory named after the skill, with `SKILL.md` at its root. The directory's name is the skill's identifier. Supporting files such as scripts, references, and seed assets live inside the same directory alongside `SKILL.md`.

Skills live directly inside their parent workflow directory. Do not use a `skills/` grouping wrapper.

Example single-skill workflow:

```text
workflows/writing/
  CONTEXT.md
  LOG.md
  writing/
    SKILL.md
```

Example multi-skill workflow:

```text
workflows/youtube-builder/
  CONTEXT.md
  LOG.md
  scripts/
  projects/
  hook/
    SKILL.md
  title/
    SKILL.md
  script/
    SKILL.md
```

Rules:

- Skill folders are exempt from the universal `CONTEXT.md` rule. The parent workflow's `CONTEXT.md` documents every bundled skill in its Contents section and Dependencies section.
- Workflows are self-contained. No cross-workflow skill references. If two workflows need the same capability, each carries its own copy or inlines the guidance.

### `wikis/`

All wikis live here. A wiki is a knowledge base: reference material, domain notes, standard operating procedures, or persistent documentation. When creating a new wiki:

- Place it in its own subdirectory under `wikis/<wiki-name>/`.
- Every wiki directory must contain a `CONTEXT.md` file.
- Every active wiki directory must contain a `LOG.md` file.

### `templates/`

Scaffolding templates for new workflows, wikis, and skills. [AI_NAME] must use these templates when creating new components so every directory stays structurally consistent.

If the `CONTEXT.md` schema changes in this file, update the templates in the same session.

### `memory/`

Durable, cross-session memory [AI_NAME] keeps about this project: feedback, references, user profile, project context, and decisions. Lives inside the repo so it travels with the project.

Read `memory/MEMORY.md` at the start of every session. Write new memories here, not to any per-user cache outside the repo.

### Other Directories

Additional top-level folders may be added over time, such as `data/`, `tools/`, or `skills/`. Every new directory must get its own `CONTEXT.md`.

## The `CONTEXT.md` Rule

Every directory [AI_NAME] creates or works in should contain a `CONTEXT.md` file. This is non-negotiable for workflow and wiki directories and strongly required everywhere else.

`CONTEXT.md` explains the purpose of the directory so future sessions and humans can understand the folder without reading every file inside it.

Required sections:

1. **Purpose** - one or two sentences stating what this directory is for.
2. **Contents** - what lives here: files, subfolders, and their roles.
3. **Usage / Trigger Conditions** - how this directory is meant to be used or invoked.
4. **Inputs** - what the workflow/component needs to run.
5. **Outputs** - what it produces.
6. **Steps** - numbered list of steps. The final step of every workflow must be: `Append an entry to LOG.md.`
7. **Dependencies** - workflows, wikis, skills, MCPs, external tools, runtime packages, accounts, and env vars.
8. **Known Issues / Gotchas** - anything that has broken before or future sessions should watch out for.
9. **Related** - pointers to related workflows, wikis, parent directories, or child directories.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flatplanet/CenterOS](https://github.com/flatplanet/CenterOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
