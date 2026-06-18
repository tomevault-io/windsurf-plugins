---
trigger: always_on
description: This project is a **Claude Code agent development lab**.
---

# CLAUDE.md — AI FIRST AGENT LAB (v2)

## Project Purpose

This project is a **Claude Code agent development lab**.
Its sole purpose is to design, build, and refine agent and skill structures that will be **reused in other projects**.

> **Important:** This repository is not a product. It is the agent infrastructure that powers other projects.

---

## Project Structure

```
siegard-code/
├── dist/              # Distribution root — production artifacts (see below)
├── docs-en/           # End-user documentation (English) for downstream projects
├── docs/              # Internal project documentation (diagrams, flow maps)
├── extras/            # Drafts, analysis notes, experimental artifacts — NOT published
├── tests/             # Test suite that validates dist/ artifacts (9 layers)
├── install.sh         # Deployment script: syncs dist/ → <target>/.claude/
├── skills-lock.json   # Locks external skill versions (analogous to package-lock.json)
└── assets/            # Static assets (logo, images)
```

### Directory rules

| Path | Purpose | Write rule |
|------|----------|------------|
| `dist/` | Published artifacts consumed by target projects | Only complete, validated artifacts |
| `docs-en/` | Human-readable docs shipped with the system | Update when commands/flows change |
| `docs/` | Internal diagrams and flow maps | Freely editable |
| `extras/` | Scratchpad: drafts, changelogs, analysis | Never referenced by agents |
| `tests/` | Automated validation of `dist/` | Must pass before promoting to `dist/` |

### install.sh

Copies and replaces files from `dist/` into `<target-project>/.claude/`. Does not remove existing files in the destination.

```bash
./install.sh <path-to-target-project>
```

### skills-lock.json

Locks the version (hash) of externally sourced skills. Update when pulling a new version of an external skill. Format mirrors a package-lock — one entry per skill with `source`, `sourceType`, and `computedHash`.

---

## 🧠 AI FIRST PRINCIPLE

This project operates under an **AI FIRST paradigm**.

> All artifacts must be designed to be consumed by agents first, and humans second.

This means:

* Prefer **structure over narrative**
* Prefer **contracts over interpretation**
* Prefer **determinism over flexibility**

---

## ⚙️ CORE RULE

> **Every agent output must be directly consumable by another agent without interpretation.**

If a human needs to interpret the output, it is incorrect.

---

## Agent Principles

Agents developed here must be:

* **Autonomous** — capable of completing tasks with minimal human intervention
* **Modular** — each skill must be independent and reusable
* **Portable** — easily importable into other Claude Code projects
* **Testable** — all behavior must be verifiable in isolation
* **Deterministic** — outputs must be predictable and schema-compliant

---

## 🧩 AI FIRST WRITING RULES

### DO

* Use **structured formats**: YAML, JSON, or strict Markdown
* Always define:

  * objective
  * input
  * constraints
  * output format
  * validation criteria
* Use **one intention per instruction**
* Use **explicit and objective language**
* Define **limits and boundaries**
* Use **controlled vocabulary**
* Return **structured failure states when needed**

Example:

```yaml
status: blocked
reason: missing_input
missing:
  - api_contract
```

---

### DON'T

* Do not write free-form text for agent communication
* Do not mix multiple intentions in a single instruction
* Do not use vague terms:

  * better
  * appropriate
  * fast
* Do not assume missing context
* Do not produce outputs outside defined schema
* Do not use conversational language:

  * please
  * if possible
* Do not make implicit decisions

---

## 📐 SPEC VS EXECUTION

### Specification Layer (Persistent)

* Defines system behavior
* Human-readable, but structured
* Includes:

  * business rules
  * domain context
  * constraints

### Execution Layer (AI Operational)

* Driven by:

  * task contracts
  * schemas
  * protocols

> Specifications are not prompts.
> They are structured context used to generate execution.

---

## 🔁 TASK MODEL (MANDATORY)

User Stories are not valid execution units for agents.

All work must be broken into **structured Tasks**:

```yaml
task:
  id: <id>
  type: <type>
  objective: <single objective>

input:
  context: <required data>

constraints:
  - <explicit rules>

output:
  format: <format>
  schema: <structure>

validation:
  criteria:
    - <objective rule>
```

---

## 🔗 AGENT COMMUNICATION

All agent-to-agent communication must:

* Use structured envelopes
* Follow predefined schemas
* Contain no free text
* Be validated before consumption

---

## 🧾 LOGGING

Logs must be:

* Structured
* Traceable
* Auditable

Never use free-form logs.

---

## Claude Code Settings

### Model

Always use `claude-sonnet-4-6` unless explicitly instructed otherwise.

### Search rules

* For any textual search, use `/ccc` before Glob/Grep (when available)

### Default behavior

* Always respond in **Brazilian Portuguese (PT-BR)** unless context requires otherwise
* Prefer objective and direct responses
* Do not restate the task before executing it
* When creating files, always check if they already exist before overwriting

### Tool usage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zig999/siegard-code](https://github.com/zig999/siegard-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
