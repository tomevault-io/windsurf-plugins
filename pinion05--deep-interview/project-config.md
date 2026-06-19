---
trigger: always_on
description: Standalone Socratic deep interview for turning vague ideas into execution-ready specs with ambiguity scoring, ontology tracking, and readiness gating.
---


# Deep Interview

## Purpose

Deep Interview is a standalone requirements-clarification skill for any autonomous agent.

It transforms vague user intent into a clear, testable, execution-ready specification by:
- asking one targeted question at a time,
- measuring ambiguity across weighted clarity dimensions,
- exposing hidden assumptions,
- tracking ontology stability across rounds,
- and refusing to proceed to execution until readiness is high enough.

This skill is runtime-agnostic. It does not depend on any specific planner, orchestrator, or execution framework.

## Use When

Use this skill when:
- the user has a vague idea and wants thorough clarification before execution,
- the task is complex enough that implementing immediately would likely cause rework,
- the user says things like:
  - "interview me"
  - "ask me everything"
  - "don't assume"
  - "make sure you understand first"
  - "I have a vague idea"
  - "help me figure out what I actually want"
- the agent needs a visible readiness gate before building.

## Do Not Use When

Do not use this skill when:
- the user already provided concrete file paths, function names, APIs, schemas, or acceptance criteria,
- the user explicitly says "just do it" or "skip the questions,"
- the task is a small direct modification,
- the user already has a sufficient PRD, plan, issue, or spec.

## Why This Exists

AI systems can generate code quickly, but vague intent still causes expensive mistakes.

The bottleneck is often not execution speed but specification quality:
- unclear goals produce wrong outputs,
- hidden assumptions create rework,
- unstable domain concepts cause architecture drift,
- missing acceptance criteria make "done" impossible to verify.

Deep Interview exists to make ambiguity visible, reduce it systematically, and produce a durable spec that any downstream agent can execute.

## Core Guarantees

This skill guarantees that it will:
- ask exactly one question per round,
- always target the weakest clarity dimension,
- explain why that dimension is the next bottleneck,
- show ambiguity scores after every round,
- track ontology stability across rounds,
- persist state for resume,
- produce a final standalone spec artifact,
- provide a runtime-neutral handoff package for downstream execution.

## Runtime Requirements

The host agent/runtime should ideally support the following capabilities.

### Required capabilities
- ask_user(question, options?)
- write_artifact(path, content)
- save_state(key, value)
- load_state(key)

### Strongly recommended capabilities
- list_files()
- read_file(path)
- search_workspace(query)
- read_workspace()

### Optional capabilities
- spawn_subagent(role, prompt) for scoped exploration
- score_with_model(prompt, temperature) for stable structured scoring
- get_timestamp()
- generate_uuid()

If no subagent system exists, the primary agent may perform exploration itself.

## Runtime-Neutral Behavior

This skill does not assume:
- a specific orchestrator,
- a specific planning framework,
- a specific folder layout,
- or a specific execution engine.

It only assumes that:
1. it can ask the user questions,
2. it can inspect the environment when relevant,
3. it can persist state,
4. and it can write a final artifact.

## Interview State Schema

Persist state under a runtime-defined key such as:

`deep-interview:<session-or-task-id>`

### State JSON

```json
{
  "active": true,
  "current_phase": "deep-interview",
  "state": {
    "interview_id": "<uuid>",
    "type": "greenfield|brownfield",
    "initial_idea": "<user input>",
    "rounds": [],
    "current_ambiguity": 1.0,
    "threshold": 0.2,
    "codebase_context": null,
    "challenge_modes_used": [],
    "ontology_snapshots": [],
    "status": "interviewing"
  }
}
```

## Project Type Detection

### Greenfield
Use greenfield when:
- there is no existing repo/workspace context,
- the request is clearly for something new,
- or codebase exploration is unavailable.

### Brownfield
Use brownfield when:
- a repo or source tree exists,
- relevant source files or manifests exist,
- or the request appears to modify or extend an existing system.

If exploration fails, default to greenfield and note the limitation.

## Brownfield Exploration Policy

Before asking the user about implementation context:
- inspect likely relevant files,
- identify existing modules, auth paths, APIs, routes, schemas, services, jobs, or UI surfaces,
- summarize findings into `codebase_context`.

Never ask the user questions that the workspace can already answer directly.

### Good
> "I found JWT auth middleware in src/auth/. Should this feature extend that path or intentionally diverge?"

### Bad
> "What authentication system does your project use?"

Facts are for the repo. Decisions are for the user.

## Clarity Dimensions

### Greenfield dimensions
1. Goal Clarity
2. Constraint Clarity
3. Success Criteria Clarity

### Brownfield dimensions
1. Goal Clarity
2. Constraint Clarity
3. Success Criteria Clarity
4. Context Clarity

### Dimension definitions

#### Goal Clarity
Can the primary objective be stated in one sentence without major ambiguity?
Are the key nouns and verbs stable?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pinion05/deep-interview](https://github.com/pinion05/deep-interview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
