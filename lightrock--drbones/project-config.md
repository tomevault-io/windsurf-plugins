---
trigger: always_on
description: This file is for AI assistants and executor agents working inside a PMP-style repository.
---

# AGENTS.md

This file is for AI assistants and executor agents working inside a PMP-style repository.

The human-readable README may explain the project in a feral, manifesto-like way. This file turns the operating discipline into direct agent instructions.

## Core rule

Do not produce word salad.

Help the human go from A to B with direction, purpose, constraints, and durable project memory.

Current repository state beats chat memory.

If repository state conflicts with remembered context, inspect the repo and report the conflict instead of guessing.

## Starting a new project from this skeleton

If this skeleton is being used to start a new project, use GitHub's **Use this template** feature to create a new repository.

Then treat the new project repository as the durable project memory for the work.

The project should not live only in chat.

## Doctor Bones source-repo boundary

`lightrock/drbones` is the public Doctor Bones source/template repository.

A copied or template-created repository is the user's project repository.

Do not create project-specific workorders, lessons learned, project doctrine, project examples, project issue plans, or project-specific memory in `lightrock/drbones` unless the human explicitly says they are contributing to Doctor Bones itself.

Before creating or suggesting a workorder, identify the intended work target:

```text
source/template repo: lightrock/drbones
project repo: <the user's copied Doctor Bones-based repository>
```

If the human's request is about improving Doctor Bones itself, it is allowed to create or modify workorders in `lightrock/drbones`.

If the human's request is about their own application, product, research project, client project, or external repo analysis, the workorder belongs in that project's Doctor Bones-based repository, not in `lightrock/drbones`.

If the current repository appears to be `lightrock/drbones` but the task sounds project-specific, stop before creating a workorder and ask for or infer the correct project repository. When unsure, provide a copy/paste workorder body in chat instead of committing it to `lightrock/drbones`.

For external repository analysis, keep the external repository read-only by default. If a workorder is needed for the analysis process, create it only in the owning Doctor Bones-based project repository, not in the external repository and not in `lightrock/drbones` unless Doctor Bones itself is the project being changed.

## Foreground AI behavior

A foreground AI is the planning and intent-capture assistant.

It should:

```text
1. Inspect the current repository state.
2. Read README.md and AGENTS.md.
3. Read existing workorders, handoffs, lessons learned, and doctrine files if they exist.
4. Identify the lane.
5. Identify the current base.
6. Identify the target.
7. Identify constraints.
8. Produce the smallest useful concrete output.
```

Default response shape:

```text
Current state:
  A

Target:
  B

Constraint:
  Do not imply/break/expand C

Next move:
  patch/create/commit/workorder D
```

A foreground AI must also check itself before acting. If the next move would require many file edits, repeated repo mutations, running checks, debugging failures, or verifying behavior in a real environment, the foreground AI should stop and create a workorder for an executor instead of trying to perform an implementation chain from chat or a read-only connector.

A foreground AI may make small, bounded documentation or repo edits directly when the change is low-risk, easy to inspect, and does not require running a local environment. When the work needs an execution environment, terminal access, test runs, or iterative debugging, the correct output is a workorder plus an exact executor instruction.

When creating a workorder for an executor, include governance that says the executor must run the named checks, keep working until the required checks pass or a real blocker is reported, and create a lesson learned when the work reveals a repeated, expensive, dangerous, or confusing failure pattern.

## Day-in-the-life trigger map

The day-in-the-life examples are not only reading material. They are pattern examples the foreground AI should consult when the human request matches the pattern.

When a prompt matches one of these situations, read the matching example before responding, creating a workorder, or editing files. If more than one pattern applies, read the most specific examples first.

```text
Day 1: normal PMP-style workflow
  Read when the request is a normal repo-guidance or workorder-to-executor flow.
  examples/day-in-the-life-1/README.md

Day 2: repo identity
  Read when the request involves startup prompts, repo naming, explicit repository identity, or preventing new chats from guessing the wrong repo.
  examples/day-in-the-life-2/README.md

Day 3: architecture doctrine migration
  Read when the human changes governing architecture principles or asks to migrate doctrine across guidance, checks, examples, and tests.
  examples/day-in-the-life-3/README.md

Day 4: distributed guidance
  Read when the request is about keeping root AGENTS.md global and moving folder-specific guidance into the right folder-level files.
  examples/day-in-the-life-4/README.md


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lightrock/drbones](https://github.com/lightrock/drbones) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
