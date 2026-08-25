---
trigger: always_on
description: This repository is **clawdForge 3.0**.
---

# CLAUDE.md

## Project Identity

This repository is **clawdForge 3.0**.

clawdForge 3.0 is an AI-native **software factory** that turns a user’s prompt into a working app or web app end to end.

It is **not**:
- a generic chatbot
- a generic agent playground
- a flashy AI demo
- a loose set of prompts

It **is**:
- a practical builder tool
- a trustworthy software factory
- a structured execution system
- a product that produces code **and** durable artifacts

Always optimize for:
- practical usefulness
- trustworthy state/progress
- strong product structure
- consistent output quality
- builder-focused UX

---

## Source of Truth

When making decisions, use the following priority order:

1. `PRD.md`
2. `Findings.md`
3. `designs/` assets and notes
4. existing code and architecture patterns already established in the repo

Do not invent a direction that conflicts with the PRD or Findings.

If something is ambiguous:
- prefer the more builder-focused interpretation
- prefer real state over theatrical UX
- prefer structured workflows over free-form prompting
- prefer durable artifacts over temporary chat output

---

## Core Product Model

clawdForge 3.0 has two primary modes:

### Interactive Mode
The system collaborates with the user before and during early planning:
- asks clarifying questions
- shapes the product direction
- creates a spec
- proposes architecture/stack
- creates an implementation plan
- gets approval at key checkpoints
- then builds mostly autonomously

### Autonomous Mode
The system:
- interprets the prompt
- uses the default stack and default policies
- minimizes interruption
- proceeds mostly autonomously
- only interrupts on true blockers, critical missing input, or high-risk decisions

Both modes should converge into the same core factory pipeline.

---

## Architectural Intent

The intended architecture is:

- **Web app** = control surface
- **Local runner** = real execution on the machine
- **LangGraph** = workflow orchestration, durable state, checkpoints, interrupts/resume
- **Claude Agent SDK** = execution engine inside workflow nodes

### Rule of Responsibility
- LangGraph decides **what happens next**
- Claude Agent SDK does **the task-level implementation work**
- the runner touches **files, shell, installs, builds, tests, git, previews**
- the UI shows and controls **the factory**

Do not collapse these responsibilities together unless there is a very strong reason.

---

## Workflow Philosophy

The system should follow a staged workflow, not “prompt in, random build out.”

Canonical flow:

1. intake
2. clarification / discovery
3. spec generation
4. architecture / stack decision
5. implementation plan
6. scaffold / foundation
7. implementation loop
8. verification loop
9. packaging / handoff

Always prefer:
- explicit stages
- visible progress
- durable artifacts
- clean handoffs between stages

Avoid giant uncontrolled agent loops.

---

## Worker Philosophy

Use a **small set of meaningful workers**.

Preferred worker types:
- Strategist
- Architect
- Planner
- Implementer
- Reviewer
- Fixer
- Packager

Workers exist for:
- context isolation
- responsibility boundaries
- tool boundaries
- execution clarity

Do **not** create an org-chart simulation with unnecessary worker roles.

Avoid agent theater.

---

## Skill Philosophy

Skills should be:
- modular
- task-specific
- dynamically loaded
- mostly invisible to the user

Good examples:
- clarify-product
- choose-stack
- write-spec
- create-plan
- scaffold-default-stack
- implement-auth
- build-ui
- run-verification
- fix-build
- package-handoff

Do not rely on one giant universal prompt when a smaller scoped skill is better.

Do not load unnecessary skills by default.

---

## UX Principles

The UX should feel:
- practical
- builder-focused
- trustworthy
- alive
- structured
- premium without being flashy

### Home Experience
The app should open into a **clean project dashboard**, not a dramatic cockpit.

### Forge Workspace
The forge workspace is the execution environment entered once a run starts.

It should:
- emphasize phase/status first
- feel immersive and builder-focused
- preserve clawdForge’s dark industrial amber-accented identity
- expose deeper developer detail in organized views

### Trust Rule
The UI must always communicate real state, not vague “AI thinking” language.

The user should be able to quickly understand:
- what is being built
- current phase
- active task
- what completed
- what is blocked
- whether intervention is needed
- whether the app currently runs

### Live Preview
A Live App / preview surface is important.
Treat preview as proof that the factory is producing something real.

---

## Design Guardrails

Use the existing designs and notes as reference.

The desired visual direction is:
- dark near-black base
- restrained amber/orange accent
- industrial / premium / practical tone
- closer to clawdForge 2.0 than generic SaaS dashboards

Do not drift toward:
- bright blue SaaS styling
- playful or gamified UI
- flat generic productivity tooling
- noisy AI-dashboard aesthetics

### Important Forge Workspace Note
The Forge Workspace is sensitive to over-refinement.
Making it “cleaner” too aggressively can remove clawdForge’s identity.

When refining that screen:
- improve hierarchy
- improve state clarity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Divici/clawdforge3](https://github.com/Divici/clawdforge3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
