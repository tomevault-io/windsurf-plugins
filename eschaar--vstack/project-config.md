---
trigger: always_on
description: You are assisting in the development of **vstack**, a VS Code–native AI engineering workflow system inspired by gstack.
---

# vstack Copilot Instructions

You are assisting in the development of **vstack**, a VS Code–native AI engineering workflow system inspired by gstack.

## Identity

vstack provides structured skills for backend/microservice development, executable via GitHub Copilot Agent Mode. It exposes six delivery roles — product, architect, designer, engineer, tester, release — plus a planner coordinator agent, each backed by hand-authored agent files that invoke skill templates.

## Core Principles

- **Template-driven install model.** Skills are Markdown templates (`src/vstack/_templates/skills/<name>/template.md` — source of truth). Agent config is in `src/vstack/_templates/agents/<name>/config.yaml` + `template.md`. Generated output is produced at install time. In the vstack source repo, do not treat `.github/` artifacts as source of truth and do not edit them directly. In downstream consumer repos that use vstack, installed `.github/` artifacts are effective project configuration and are usually committed.
- **VS Code native.** Skills run in Copilot Chat / Agent Mode. No assumptions about Claude Code or CLI-only flows.
- **Backend first.** Prioritize API correctness, reliability, observability, CI/CD, contracts, performance, and security. Browser automation is optional and pluggable.

## Artifact Choice Policy

- Agents are for roles and handoffs.
- Skills are for reusable procedures.
- Instructions are for always-on policies.
- Prompts are for one-shot task framing.
- Hooks are for repository-level automation.

When proposals span multiple artifact types, split them into small independent changes.

## Reasoning & Critical Thinking

Evidence-based reasoning is available as an explicit opt-in framework via `prompt/reasoning`.

When `prompt/reasoning` is invoked, apply these principles:
- **Truth > agreement** — evaluate correctness, not user preference
- **Evidence > opinion** — support claims with facts, logic, or explicit assumptions
- **Critical thinking > people-pleasing** — challenge premises before accepting them

For structured adversarial review workflows, agents invoke `skill/advise`, which can reference `prompt/reasoning` when deeper critical analysis is needed.

**For downstream users:** Add `prompt/reasoning` to your vstack installation (in your `.vstack/vstack.json` manifest or via your preferred vstack distribution mechanism), then run `vstack install` to generate `.github/prompts/reasoning.prompt.md` in your repository. Agents and skills can invoke it by reference.

## System Structure

```
src/vstack/      ← Python package (source of truth)
src/vstack/_templates/
├── skills/<name>/
│   ├── template.md             ← skill body (edit these)
│   └── config.yaml
├── skills/_partials/           ← shared partial snippets
├── agents/<name>/
│   ├── template.md             ← agent instructions body
│   └── config.yaml             ← agent frontmatter fields
├── instructions/<name>/
│   ├── template.md             ← instruction file body
│   └── config.yaml
└── prompts/<name>/
    ├── template.md             ← prompt file body
    └── config.yaml
docs/            ← overview.md, design.md, skills.md, workflow.md, roadmap.md, adr/
.github/         ← generated output (never edit directly*)
├── skills/<name>/SKILL.md
├── agents/<name>.agent.md
├── instructions/<name>.instructions.md
└── prompts/<name>.prompt.md
.vstack/         ← project-scope vstack state and config
└── vstack.json                 ← install manifest (generated)
```

*Hand-authored exceptions in `.github/`: `copilot-instructions.md`, `CODEOWNERS`,
`PULL_REQUEST_TEMPLATE/`, `ISSUE_TEMPLATE/`, `workflows/`.

Generated files are written to `.github/` at install time:

```bash
python3 -m vstack install
```

## In-repo Installation

When vstack is installed inside its own repo (`.github/agents/` and `.github/skills/` exist):

**Source of truth is always in `src/vstack/_templates/`.** After every change:

| What changed | Command to run |
|---|---|
| `src/vstack/_templates/agents/<name>/config.yaml` or `template.md` | `python3 -m vstack install` |
| `src/vstack/_templates/skills/<name>/template.md` or `_partials/*.md` | `python3 -m vstack install` |
| `src/vstack/_templates/instructions/<name>/template.md` | `python3 -m vstack install` |
| `src/vstack/_templates/prompts/<name>/template.md` | `python3 -m vstack install` |

A single command handles all artifact types:

```bash
python3 -m vstack install
```

Never edit `.github/agents/`, `.github/skills/`, `.github/instructions/`, or `.github/prompts/` directly — changes will be overwritten.

## Execution Model

**Option A (current):** Single Copilot context window. A role agent reads the relevant skill and executes in one call.

**Option B (future):** Sequential multi-agent pipeline — `[vision → architecture → verify → release]`. Each stage is a separate model call; output artifacts feed the next stage. Design all config formats and generators to support this with minimal refactoring.

## Verification (Microservices-First)

`verify` and `verify-lite` must cover:

- Unit, integration, and contract tests (OpenAPI / JSON Schema / Protobuf)
- API smoke tests, migrations, retries/backoff, timeouts, circuit breakers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eschaar/vstack](https://github.com/eschaar/vstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
