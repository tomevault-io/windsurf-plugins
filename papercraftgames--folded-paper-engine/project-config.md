---
trigger: always_on
description: - The **user prompt is the authoritative starting instruction** for this run.
---

# Repository Guidelines

## Agent Startup

- The **user prompt is the authoritative starting instruction** for this run.
- Open `planning/` early. The active plan is the file directly under `planning/` (not inside `planning/complete/`).
- If multiple active plans exist, ask the user which to run.
- If no active plan exists:
  - If the user prompt contains a plan, checklist, or task list: **save it as a new active plan file in `planning/`**, then proceed.
  - If the user prompt describes work but is not already a plan: create a brief plan in `planning/` (goals + checklist), then proceed.
  - Only ask where work is tracked if the user prompt provides insufficient detail to create a plan.

---

# Project Structure & Module Organization

- `src/` holds the primary TypeScript source used to generate parts of the Blender add-on and supporting tooling.
- Generated outputs may include Python files used by the Blender add-on and supporting runtime metadata.
- The Blender add-on defines custom panels and properties used to mark gameplay-relevant objects inside Blender scenes.
- Exported scenes (typically GLTF/GLB) carry these properties into Godot where the Folded Paper Engine runtime interprets them.

Key components may include:

- Blender-side logic (Python add-on code)
- TypeScript tooling that generates or supports add-on/runtime code
- Godot-side runtime scripts (GDScript) that interpret exported scene data
- Documentation and demo assets explaining the Blender → Godot workflow

---

# Build, Test, and Development Commands

- `yarn build` compiles the TypeScript tooling used to generate parts of the Blender add-on or supporting assets.
- Some build steps may generate Python code for the Blender add-on or supporting metadata for the Godot runtime.

Development typically involves three environments:

- **Blender** for authoring scenes and assigning gameplay properties.
- **TypeScript tooling** for generating or maintaining add-on/runtime code.
- **Godot** for importing scenes and running gameplay logic.

When modifying Blender integration or export behavior, verify the workflow:

1. Mark objects in Blender using the Folded Paper Engine panels.
2. Export the scene to GLTF/GLB.
3. Import into Godot and confirm gameplay elements behave correctly.

---

# Coding Style & Naming Conventions

- This project uses multiple languages:

  - **TypeScript** for tooling and code generation.
  - **Python** for the Blender add-on.
  - **GDScript** for the Godot runtime.

- Follow existing style conventions within each language area.
- Use consistent naming between Blender property names and the Godot runtime that consumes them.
- When modifying exported property names or schemas, update both Blender and Godot code paths together.

- TypeScript formatting follows the conventions already used in the repository (2-space indentation and double quotes unless the file shows otherwise).

---

# Testing Guidelines

Testing in this project is primarily workflow-based rather than unit-test based.

When making changes, validate the full pipeline:

1. Create or modify an object in Blender.
2. Assign Folded Paper Engine properties using the Blender panel.
3. Export the scene to GLTF/GLB.
4. Import into Godot and verify that gameplay elements behave correctly.

Focus especially on:

- Object type recognition (players, triggers, speakers, etc.)
- Property propagation from Blender to Godot
- Runtime behavior inside Godot scenes

---

# Commit & Pull Request Guidelines

- Commit messages follow a Conventional Commits pattern such as `feat:`, `doc:`, or `chore:` with optional scopes.
- Keep subjects imperative and concise; automated commits may appear as `chore: (repo) Automatic commit`.
- PRs should include a clear summary and screenshots or demo scenes when relevant.
- Note any testing or validation performed in the PR description.

---

# Agent Workflow & Progress Tracking

- Treat the **user prompt** as the authoritative scope for this run; do not down-scope without explicit user approval.
- Treat the `planning/` directory as the authoritative persisted work state once a plan exists. The active plan is the file directly under `planning/` (not inside `planning/complete/`).

- Work in **phases**:

  - At the start of a run, identify the next achievable group of checklist items (a “phase”) from the current plan.
  - Phases are **sequential by default** and form a single ordered queue.
  - Do NOT treat later phases as alternatives unless the plan explicitly marks them as optional or branching.
  - A phase should be sized to complete cleanly in the current run without guesswork or scope changes.
  - If a phase is too large or contains uncertainty, split it and proceed with the smallest clearly-achievable subset.

- Default to **forward progress**:

  - **Plan order is mandatory.**
  - Execute checklist items strictly in plan order whenever possible.
  - Do NOT present alternative next steps or choices when the next plan item is clear.
  - Do NOT ask whether to continue when unchecked plan items remain.
  - Do NOT present numbered or bulleted “Next steps” lists.
  - End updates with a single `Next:` line stating the immediate next planned action in plan order.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [papercraftgames/folded-paper-engine](https://github.com/papercraftgames/folded-paper-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
