---
trigger: always_on
description: You are working on an OpenTide (Open Threat Informed Detection Engineering) repository.
---

# AGENTS.md - Agent Instructions for OpenTide Repository

You are working on an OpenTide (Open Threat Informed Detection Engineering) repository.

OpenTide is a framework that enables the creation of YAML files called Objects, which model the detection engineering lifecycle. OpenTide manages the complete corpus of knowledge around threats and detection research, alongside detection rules deployment in an as-code manner.

## Core Principles

## Prime Directives
- _Correctness_ : You must be exact, and cognitively correct as to how you break down intelligence and complex threat data into modelled OpenTide data.
- _Consistency_ : You must maintain a consistent OpenTide repository. Sometime, it's better to improve several objects before creating a new one to correctly ingest a change in perspective.
- _Transparency_ : The changes, decisions, proposals you perform must all be rationalized, discoverable and shown to the user. 
- _Critcal Thinking_ : No vague conclusion, all modelling decision must be critical. You are allowed to disagree with the user, when there is a poor reasoning as to the initial directive.
- _Autonomous_ : As much as possible, you aim at performing end-to-end changes. 


### Guardrails
**Only modifies content files, not framework files**
- **NEVER** Tamper with schemas or templates
- **DO NOT** Change configuration files except if explicitely instructed
- **NEVER** Creating new folders or restructuring the repository
- **ALWAYS** Avoid calling non IDE or simple web search tools, unless deemed strictly necessary - if so, interrupt and ask the user.

**Enforce Top Down**
- Given threat intelligence, you create threat object(s).
- Given threat objects, you create detection objective(s).
- Given Objective(s), you create Rule.
- Per run, you avoid mixing object types. You always generate the right object type, and if prompted, you ask if doing it in another MR/PR (and after object is merged) is not preferable. Mixing Object Types is NOT the intended workflow.

### Project Structure Integrity
- **NEVER** create new folders unless explicitly requested
- **NEVER** Generating or modifying UUIDs for existing objects
- **ALWAYS** respect the existing folder structure
- **ALWAYS** validate files against JSON schemas before saving
- **ALWAYS** use templates as a guide for object structure

## OpenTide Framework Concepts

OpenTide structures the Detection Engineering lifecycle as as-code (YAML) objects managed in a git repository. There are 3 core object types:

### 1. Threat Vectors (TVM)
- **Purpose**: Represent atomically defined TTPs (Tactics, Techniques, Procedures) at a low level
- **Source**: Directly generated from threat intelligence
- **Chaining**: Allows to interrelate TVMs to one another (bi-directionally) to represent an attack path
- **Abbreviation**: TVM
- **Location**: `Objects/Threat Vectors/*.yaml`

### 2. Detection Objectives (DOM)
- **Purpose**: Represent detection capabilities for identified threats
- **Relationship**: Support 1:N and N:1 relations with Threat Vectors
- **Components**: Composed of signals (atomic detection rule ideas). Signals can be referenced by Detection Rules.
- **Abbreviation**: DOM
- **Location**: `Objects/Detection Objectives/*.yaml`

### 3. Detection Rules (MDR)
- **Purpose**: Detection-as-Code files for deployment
- **Relationship**: Directly linked to Detection Objectives, indirectly to Threat Vectors
- **Target**: Can target any configured detection system
- **Abbreviation**: MDR
- **Location**: `Objects/Detection Rules/*.yaml`

## Repository Structure

```
.
├── Objects/
│   ├── Threat Vectors/     # TVM files (*.yaml)
│   ├── Detection Objectives/  # DOM files (*.yaml)
│   └── Detection Rules/    # MDR files (*.yaml)
├── Schemas/
│   ├── *.json              # JSON Schema definitions for each object type
│   └── Templates/
│       └── *.yaml          # YAML templates showing object structure
└── Configurations/
    └── systems/
        └── *.toml          # System configuration files
```

## Workflow Guidelines

### When Creating Objects from Intelligence Reports

1. **Analyze the Intelligence (if presented)**
   - Review the provided intelligence thoroughly
   - Identify distinct TTPs and detection opportunities
   - Map relationships between threats and detection capabilities
   - **Avoid infering from intuition of knowledge that are not initially present in the intelligence**
      > When there is not enough data from the intelligence presented, you allowed to propose alternative but **STOP** from proceeding before user accepts the plan.
   - **STOP** and ask the user for precision, if you are unsure of which type of objects you are supposed to create.

2. **Plan the Object Hierarchy**
   > Important: The following is the default hierarchy. Based on user prompting, you may need
   > to focus on specific object types at a time, or generate the entire structure in one go.
   - Determine which Threat Vectors (TVMs) need to be created and chaining opportunities
   - Identify corresponding Detection Objectives (DOMs)
   - Plan Detection Rules (MDRs) if applicable

4. **Check for Existing Content**
   - Search the repository for related objects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opentidehq/ShareTide](https://github.com/opentidehq/ShareTide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
