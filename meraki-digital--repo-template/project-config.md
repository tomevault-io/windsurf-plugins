---
trigger: always_on
description: Purpose: A single, repo-agnostic control file that defines always-on commands, style, and structure for agentic work.
---

# agents.md (Base)

Purpose: A single, repo-agnostic control file that defines always-on commands, style, and structure for agentic work.
Extensible by loading supplemental rule files so each repo can add commands without editing this base.

Load order and overrides:
1. `./agents.md`  (this file)
2. `./agentic/agents.d/*.md`  (repo-specific rules, optional)
3. `./agents.local.d/*.md`  (developer or machine specific, git-ignored, optional)

If two files define a command with the same `name`, the last loaded definition wins.

---

## Global defaults

- Timezone: America/Chicago
- Style: plain, direct, no em dashes
- Encoding: UTF-8
- Line endings: LF
- Confirm only on destructive actions
- Use ISO timestamps `YYYY-MM-DD HH:MM`
- Do not create new folders from this base file

---

## Folder expectations

This base file assumes only these paths exist by default:
- `/` root with `agents.md`
- `./agentic/agents.d/`
- `./agents.local.d/`
- `./agentic/tasks/`
- `./agentic/tasks/explainers/`
- `./agentic/tasks/mods/`  (contains per-module folders like `0001`, `0002`, etc.)

Repo-level files in `agentic/agents.d/` may introduce additional folders if that repo needs them.

---

## Command schema

Each command must follow this schema for machine parsing and human readability:

```
name: <Trigger>
intent: <One-line purpose>
inputs: <Questions or auto-detection rules>
writes: <Files created or modified>
steps:
  - <Action 1>
  - <Action 2>
done: <What to return to chat>
```

Rules:
- `name` must be unique, case insensitive
- If `inputs` is empty, run immediately
- Never delete user content unless the command states to do so
- Only write to files under the existing folders listed above

---

## Development workflow commands

### 1) Do Step 1
```
name: Do Step 1
intent: Run discovery using ./agentic/tasks/01-discover-requirements.md as instructions
inputs:
  - Determine target mod folder with this precedence:
      1) If user explicitly specifies one in the current message, use that
      2) Else if a mod folder was established earlier in this conversation thread, use that
      3) Else ask: "Which mod folder under agentic/tasks/mods should I work on? (example: 0001)"
  - Ask: "Would you like the Discovery Interview A) Inline or B) File-Based?"
      * If inline: Conduct interview in chat
      * If file-based: Create ./agentic/tasks/mods/<mod-folder>/interview/ folder and write all questions to a single file named srs-discovery-interview.md
writes: ./agentic/tasks/mods/<mod-folder>/<mod-folder>-srs-executive-[project-name].md, ./agentic/tasks/mods/<mod-folder>/<mod-folder>-srs-technical-[project-name].md, ./agentic/tasks/mods/<mod-folder>/interview/ (if file-based)
steps:
  - Read ./agentic/tasks/01-discover-requirements.md
  - Check if mod folder and seed file (e.g., <mod-folder>-seed.md) exist
  - Ask user for interview format preference (inline or file-based)
  - If file-based: Create ./agentic/tasks/mods/<mod-folder>/interview/ folder and write all discovery questions to srs-discovery-interview.md
  - If seed file is present, read it as the initial concept and conduct discovery interview to clarify any gaps
  - If seed file is not present, conduct full discovery interview to gather initial concept
  - Generate Executive SRS and Technical SRS documents
  - Write both SRS files to ./agentic/tasks/mods/<mod-folder>/ with ISO timestamps in headers
  - Cross-reference the documents in their appendices
done: Return short bullet summaries of both SRS documents and their saved file paths
```

### 2) Do Step 2
```
name: Do Step 2
intent: Create a PRD using
  - ./agentic/tasks/02-create-prd.md as instructions
  - ./agentic/architecture.md (if exists) to understand established patterns and tech stack
  - ./agentic/tasks/mods/<mod-folder>/<mod-folder>-srs--executive-[project-name].md as context
  - ./agentic/tasks/mods/<mod-folder>/<mod-folder>-srs--technical-[project-name].md as context
inputs:
  - Resolve mod folder using the same precedence as Do Step 1
writes: ./agentic/tasks/mods/<mod-folder>/<mod-folder>-prd-[project-name].md
steps:
  - Generate a complete PRD for the chosen mod folder
  - Include goals, non-goals, user stories, acceptance criteria, risks, and open questions
  - Ensure PRD aligns with patterns and conventions from architecture.md
done: Return the PRD title, section list, and file path
```

### 3) Do Step 3
```
name: Do Step 3
intent: Generate a task list using
  - ./agentic/tasks/03-generate-tasks.md as instructions
  - ./agentic/tasks/mods/<mod-folder>/<mod-folder>-prd-[project-name].md as context
inputs:
  - Resolve mod folder using the same precedence as Do Step 1
writes: ./agentic/tasks/mods/<mod-folder>/<mod-folder>-tasks-[project-name].md
steps:
  - Produce a structured list with IDs, owners (placeholder if unknown), estimates, dependencies, and acceptance criteria
done: Return task count, critical path items, and file path
```

### 4) Do Step 4
```
name: Do Step 4
intent: Produce a tech-stack and associated explainers using
  - ./agentic/tasks/04-explainer.md as instructions
  - ./agentic/tasks/mods/<mod-folder>/<mod-folder>-srs-technical-[project-name].md as context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meraki-digital/repo-template](https://github.com/meraki-digital/repo-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
