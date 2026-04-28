---
trigger: always_on
description: These mandates are foundational. All agents working in this project MUST adhere to these rules without exception.
---

# AIM v2.2 Core Mandates

These mandates are foundational. All agents working in this project MUST adhere to these rules without exception.

## 1. File Format & Extension
- **Extension:** Every AIM artifact MUST end in `.intent`.
- **No Substitutes:** Never produce YAML, JSON, XML, or Markdown in place of an `.intent` file.
- **Header:** Every file must start with exactly: `AIM: <component>#<facet>@2.2`
- **Identity:** The component and facet in the header MUST match the filename and directory path.

## 2. Syntax Rules
- **Blocks:** Use UPPERCASE keywords and braces (e.g., `INTENT Name { ... }`).
- **Assignments:** Use `KEY: value`.
- **Lists:** Use hyphen-led entries (`- "item"`).
- **Prose:** Always quote natural language values.
- **No Commas:** Commas are not required between entries.

## 3. Role Dispatch
- **@aim-registry**: Registry -> Local Materialization.
- **@aim-author**: Requirements -> Intent.
- **@aim-implementer**: Intent -> Code/Tests.
- **@aim-verifier**: Code -> Drift Report.
- **@aim-repairer**: Drift -> Alignment.

---

# Application Intent Model (AIM) v2.2

Application Intent Model (AIM) is an intent-driven specification language for humans and AI agents. It captures product behavior in a form that is readable enough for product and design discussion and structured enough for implementation, verification, repair, and deterministic synthesis.

AIM supports progressive precision:

- start with a single intent envelope
- add precision facets only where stronger guarantees are needed

This keeps simple components easy to author while allowing high-fidelity implementation and synthesis for complex systems.

AIM is meant to read like a coordination document first and a formal specification second. In practice, many components will stay as a single `.intent` file with a few embedded details. The stricter identity, resolution, and validation rules exist so tools can agree on what a component means once teams need automation, verification, or synthesis.

---

## 1. Core Model

A component is identified by a namespace such as `company.subsystem` (for example `game.snake`, `juice.tasks`).

Each component has:

- one required intent envelope
- zero or more optional precision facets: `schema`, `flow`, `contract`, `persona`, `view`, `event`
- zero or more optional mapping files using facet `mapping`

The intent envelope is the canonical entrypoint for a component. All other detail is attached to that entrypoint directly or indirectly.

### 1.1 AIM As A Coordination Layer

AIM is the authoritative shared artifact between humans and coding agents.

It describes intended system behavior, structure, and constraints in a form that multiple agents can read and act on consistently. AIM is not an agent protocol, agent topology, or swarm configuration language. The number of agents involved in a workflow is outside the language and may vary by project, task, or implementation strategy.

The intent envelope is the primary behavioral reference for a component. Precision facets add detail and stronger guarantees, but they do not replace the authority of the intent envelope.

### 1.2 How Agents Should Treat Intent

One or more agents may participate in any of the following responsibilities:

- interpret requirements by creating or refining intent
- implement code and tests by reading intent and available facets
- validate code, tests, and outputs against intent
- repair mismatches by updating code or, when necessary, revising intent explicitly

Agents should coordinate primarily through AIM artifacts rather than relying on unstated assumptions, private memory, or ad hoc chat context.

Normative guidance:

- implementation should not invent material behavior absent from intent
- when detail is missing, agents should preserve the documented intent and minimize assumptions
- when assumptions are necessary, they should be surfaced for review or converted into explicit intent updates
- when implementation and intent disagree, the mismatch must be resolved explicitly rather than silently normalized
- if the implementation is wrong, fix the code
- if the intent is outdated, revise the intent before continuing implementation
- if requirements changed, update intent before further coding or repair

### 1.3 Agent Operating Roles

The following roles are defined as operational guidance for systems and teams using AIM. They are not AIM language constructs and do not appear in `.intent` source files.

A workflow may use one agent, many agents, or any combination of these roles. One agent may perform multiple roles, and multiple agents may share one role.

#### Intent Author

Purpose:

- interpret requirements and translate them into AIM artifacts

Reads:

- product requirements
- existing AIM files
- relevant code and tests when refining an existing system

Writes:

- intent envelope
- precision facets when needed
- explicit clarifications of assumptions or ambiguities

Primary goal:

- make intended behavior explicit, reviewable, and implementation-ready

Must do:

- express requirements in intent rather than leaving them implicit
- add facets only when they increase useful precision
- surface ambiguity when requirements are incomplete or conflicting

Must not do:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juicejs/application-intent-model](https://github.com/juicejs/application-intent-model) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
