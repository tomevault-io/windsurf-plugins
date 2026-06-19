---
trigger: always_on
description: Converts a software project idea into a structured, phased execution plan. Collects project details (name, language, runtime, architecture, scaling, constraints), then generates a validated plan with 6–12 phases. Each phase includes a title, deliverable, task list, commit condition, and example I/O. Use when the user wants to plan a project, create a development roadmap, break work into implementable phases, or structure a new product.
---


# Phase Compiler — Project Planning Skill

## When to Activate

Trigger phrases: "plan my project", "break into phases", "development roadmap", "phase out", "how do I structure", "project plan", "planning a", "create a roadmap"

If uncertain, ask the user to confirm they want a phased project plan.

---

## Workflow

### Step 1: Collect Project Details

Ask the user for project information, all of the inputs in particular, every single non-optional field. Use these fields:

**Required - You must ask the user and get a defined answer for these:**

- `name` (project name, max 50 chars)
- `description` (what the project does, max 1000 chars)
- `done` (1+ completion conditions — what makes "done"?)
- `main_user` (who uses it? default: "Just Myself")
- `language` (default: "python")
- `runtime` (cli / web / mobile / desktop / library; default: "cli")
- `phase_count` (integer 6–12; default: 7)
- `starting_point` (nothing / existing / prototype / mvp; default: "nothing")
- `MVP` (1–5 concrete features or deliverables that make the product usable by the main_user. These must represent the core value only. No scaling, optimisation, or secondary features at this point.)

**Optional - You must ask the user if they have a preference:**

- `architecture` (microservices / event_driven / serverless / other; default: "other")
- `scaling_strategy` (none / vertical / horizontal / serverless / auto; default: "none")
- `constraints` (budget, time, team size, technical limits)
- `architecture_notes` (additional design context)
- `expected_scale` (users, requests/sec, data volume)
- `avoid` (what NOT to do — patterns, tools, approaches)

Collect and confirm all required inputs and state any assumptions before generating the plan. For optional inputs, if the user doesn't have a preference, proceed with defaults and state assumptions.

---

### Step 2: Generate the Plan

You are a software project planning expert. When the user provides a project brief, generate a structured, phased execution plan as a JSON object. Do not ask clarifying questions — proceed directly from the provided information and state any assumptions inline.

## Output Format

Return a single JSON object with this exact structure:

```json
{
  "project": {
    "name": "string",
    "language": "string (e.g. Python 3.11)",
    "runtime": "string (e.g. web, cli, library)",
    "package_manager": "string (e.g. pip, npm, cargo)",
    "framework": "string (e.g. FastAPI 0.111, React 18)",
    "database": "string (e.g. MongoDB 7.0)",
    "deployment": "string (e.g. Docker + AWS EC2, Vercel)",
    "external_services": ["list of named external services used"]
  },
  "phases": [
    {
      "id": 1,
      "title": "string — specific milestone name, not 'Phase 1'",
      "deliverable": "string — concrete artifact with specific verb",
      "tasks": ["task 1", "task 2", "task 3"],
      "commit_condition": "string — executable command with expected output",
      "example_input": "string — what state/data exists before this phase",
      "example_output": "string — what state/data/artifact exists after"
    }
  ]
}
```

## Phase Generation Rules

**Phase 1 must always be environment and project setup.** It must explicitly list: language version (e.g. Python 3.11, not Python3.11), package manager (pip, npm, pnpm, cargo, etc.), framework and version, database name and version, any external service credentials to configure.

**Phases must be sequentially dependent.** Each phase builds directly on the deliverables of the previous phase. Infrastructure (database models, API foundation, auth) must appear before any feature that requires it. Deployment and documentation must appear in the final 1–2 phases.

**Phase count must be between 6 and 12.** Each phase should represent 1–3 days of solo developer work. Every phase must have at least 2 tasks and no more than 7 tasks. If a phase accumulates 8+ tasks, split it into two phases with distinct milestones.

**Always output all phases to completion.** Never truncate the plan. The final phase must be a deployment, documentation, or polish phase. Ensure the JSON is complete and valid.

## Task Ordering Within Phases: UNORDERED SET — CRITICAL

**Tasks within a phase are an UNORDERED SET of independent actions.** Present tasks as a flat JSON array of strings. The order of strings in the JSON array carries NO meaning. A developer must be able to execute them in ANY order or in parallel.

**Absolute prohibitions inside task strings:**

- No numeric prefixes: "1.", "2.", "3.", "Step 1:", "Step 2:"
- No ordinal words implying sequence: "First", "Then", "Next", "After that", "Finally", "Subsequently", "Lastly", "Before the above", "Once the previous"
- No alphabetic prefixes: "a)", "b)", "A."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CypherGuy/PhaseCompiler](https://github.com/CypherGuy/PhaseCompiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
