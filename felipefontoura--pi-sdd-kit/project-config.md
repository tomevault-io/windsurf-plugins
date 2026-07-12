---
trigger: always_on
description: This repository contains `@felipefontoura/pi-sdd-kit`, a Pi package that provides skills and templates for a practical Specification-Driven Development workflow.
---

# AGENTS.md

This repository contains `@felipefontoura/pi-sdd-kit`, a Pi package that provides skills and templates for a practical Specification-Driven Development workflow.

## Repository Purpose

The package helps Pi users run an SDD workflow with explicit artifacts, human gates, traceability, and verification.

Additional study/reference repositories:

- https://github.com/felipefontoura/sdd-book
- https://github.com/compozy/compozy/tree/main/skills

Use `sdd-book` as conceptual study material for how the SDD workflow should be explained and used.

Use the Compozy skills as an inspiration for quality bar: technical rigor, explicit gates, verification discipline, review structure, controlled scope, and operational clarity. Do not vendor or copy content from either repository unless explicitly requested.

Primary workflow:

```text
IDEA → PLAN → PRD → SPEC → TASKS → EXEC → REVIEW
```

The public Pi skill commands are intentionally prefixed with `sdd-`:

```text
/skill:sdd-init
/skill:sdd-steering
/skill:sdd-idea
/skill:sdd-plan
/skill:sdd-prd
/skill:sdd-spec
/skill:sdd-tasks
/skill:sdd-exec
/skill:sdd-review
/skill:sdd-status
```

Do not reintroduce `/sdd:*` extension aliases unless explicitly requested.

## Language Policy

Repository source, README, skills, templates, and package documentation should be written in **EN-US**.

Runtime behavior documented by the skills is different:

- skills should respond in the user's initial chat language;
- skills should write generated project artifacts in the user's initial chat language;
- paths, file names, status values, commands, IDs, and code symbols remain canonical and should not be translated.

## Important Directories

```text
skills/
  _shared/references/
    sdd-practical.md      # Shared SDD rules, gates, status values, language policy
    templates.md          # Compact internal templates used by skills
  sdd-init/SKILL.md
  sdd-steering/SKILL.md
  sdd-idea/SKILL.md
  sdd-plan/SKILL.md
  sdd-prd/SKILL.md
  sdd-spec/SKILL.md
  sdd-tasks/SKILL.md
  sdd-exec/SKILL.md
  sdd-review/SKILL.md
  sdd-status/SKILL.md

templates/
  README.md
  sdd-index.md
  sdd-workflow.md
  steering-product.md
  steering-tech-stack.md
  steering-conventions.md
  steering-landing-page.md
  steering-principles.md
  idea.md
  plan.md
  requirements.md
  design.md
  tasks.md
  task.md
  review.md
  handoff.md
  issue.md
  adr.md
```

## Package Manifest

`package.json` declares this as a Pi package:

```json
"pi": {
  "skills": [
    "./skills"
  ]
}
```

The npm package must include:

```json
"files": [
  "skills",
  "templates",
  "README.md",
  "AGENTS.md",
  "LICENSE"
]
```

There should be no `extensions` entry unless the user explicitly requests real Pi extensions.

## Skill Naming Rules

Each skill directory must contain `SKILL.md`.

The frontmatter `name` must exactly match the directory name:

```yaml
---
name: sdd-prd
description: '...'
---
```

Descriptions should be quoted with single quotes to avoid YAML parse errors when using colons.

Good:

```yaml
description: 'Creates requirements: stories, criteria, and boundaries.'
```

Bad:

```yaml
description: Creates requirements: stories, criteria, and boundaries.
```

## SDD Artifact Structure Expected by Skills

Generated artifacts should live in user projects under:

```text
.ai/
  strategy/
    handoff/
      strategy-brief.md # optional upstream input from strategy packages
  steering/
    product.md
    tech-stack.md
    conventions.md
    principles.md
    *.md
  sdd/
    INDEX.md
    WORKFLOW.md        # optional workflow guide
    PLAN.md
    handoff/
      sdd-brief.md      # downstream SDD output contract
    ideas/
      001-feature-idea.md
    specs/
      001-feature-name/
        .status
        requirements.md
        design.md
        tasks.md
        review.md
        decisions.md
```

Do not revert this to `.sdd/`.

## Status Values

Valid `.status` values are:

```text
idea:exploring
idea:captured
plan:draft
plan:approved
requirements:draft
requirements:approved
design:draft
design:approved
tasks:draft
tasks:approved
implementation:in-progress
implementation:done
review:done
```

## Gate Rules

The SDD flow depends on human approval gates:

- Draft artifacts may be saved before approval.
- Draft artifacts do not unlock the next phase.
- `design.md` requires approved requirements unless explicitly a non-binding draft spike.
- `tasks.md` requires approved design unless explicitly draft planning.
- implementation requires `tasks:approved`.
- review should validate against requirements, design, tasks, and verification evidence.

## Traceability Rules

Use stable IDs:

```text
US-001
FR-001
NFR-001
TD-001
T1
```

Templates should preserve lightweight traceability:

- requirements → design;
- requirements → tasks;
- requirements/tasks/design → review;
- requirements/tasks/design/review → `.ai/sdd/handoff/sdd-brief.md`.

Avoid heavy bureaucracy for small features, but do not remove traceability entirely.

## Template Policy

There are two template layers:

1. `skills/_shared/references/templates.md`
   - compact internal reference loaded by skills;
   - should remain concise.

2. `templates/*.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipefontoura/pi-sdd-kit](https://github.com/felipefontoura/pi-sdd-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
