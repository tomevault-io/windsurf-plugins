---
trigger: always_on
description: The file `.github/ProjectDescription.md` contains a comprehensive description of the project architecture, communication software structure, library blocks, protocols, and data flow.
---

# Copilot Instructions

## Project Description

The file `.github/ProjectDescription.md` contains a comprehensive description of the project architecture, communication software structure, library blocks, protocols, and data flow. 

The functional description of automation systems should provide the developer with a clear and detailed understanding of what the system is supposed to do and what are the requirements to achieve this. It should also provide guidance on how the system should be designed and implemented to effectively meet these requirements.


**Rule:** The project description should include:

1. General description of the system.
2. Detailed functional requirements for devices controlled by the automation system taking into account the technological division into objects (e.g. pump, valve, measurement) or groups of objects with the full structure of nesting objects, the so-called “plant hierarchy”.
3. The object description should include:
   •	a brief description of the operation of the facility,
   •	operating modes,
   •	in automatic mode, the algorithm for developing commands controlling an object (if applicable to a single object or a reference to the description of control of a larger group of objects),
   •	description of interlocks with logic (e.g. for start, operation, opening, closing etc.)
   •	handling of errors and exceptions,
   •	control method with HMI
4. The automatic operation of a group of objects should be described with the same requirements as for individual objects. In addition, the object control algorithm should clearly describe how the system is to be controlled (settings, emergency situations, calculations, performance requirements, control loops etc.)
5. In case of sequences of activities, the following should be described:
   •	starting conditions, interruptions of sequences with logic,
   •	description of sequence steps with the commands for a given step,
   •	conditions for transitions between steps with logic,
   •	the sequence should be represented graphically.
6. Data exchange interfaces between systems

**Always** consult this file for context when answering questions about the project.

**Rule:** When making changes that affect the project structure, architecture, communication logic, block hierarchy, message definitions, or data block layout, **always** update `.github/ProjectDescription.md` to reflect those changes. This includes:

- Adding, removing, or renaming program blocks (FB, FC, OB, DB, UDT)
- Changing message IDs, data structures, or communication channels
- Modifying the call hierarchy or adding new communication channels
- Changes to library blocks

**Rule:** If `.github/ProjectDescription.md` is empty or does not exist, and you are analyzing the project's software structure (e.g. reviewing program blocks, communication logic, library blocks, or data flow), **always** generate and populate `ProjectDescription.md` with a comprehensive description of the project before proceeding.

**Rule:** When creating or updating `ProjectDescription.md`, **always** include **Mermaid diagrams** (`\`\`\`mermaid`) to visually illustrate key aspects of the project. At minimum, include diagrams for:

- **Architecture overview** — block/folder dependencies and relationships (use `graph TD`)
- **Communication channel topology** — TCP connections, OPC UA channels, ports (use `graph LR`)
- **Call hierarchy** — OB → FB → library instance call tree (use `graph TD`)
- **Data flow / sequence diagrams** — message send/receive sequences for each communication direction (use `sequenceDiagram`)
- **State machines** — for blocks with complex internal state logic (use `stateDiagram-v2`)
- **Data structures** — for key UDT/message layouts (use `packet-beta` or `block-beta`)
- **Queue/scheduling logic** — for FIFO managers or priority systems (use `flowchart LR`)

Place each diagram **directly above or below** the corresponding textual description it illustrates. Keep text descriptions alongside diagrams — diagrams supplement, not replace, the text.

---

## Project Directories

### `Tools/`

Directory with utility scripts for the project (Python, PowerShell, etc.). Each script has a corresponding `.md` file with a description, usage examples, and dependencies.

**Rule:** When creating a new script, **always** place it in `Tools/` and create a matching `.md` description file (e.g. `Tools/_myScript.py` + `Tools/_myScript.md`).

### `UserFiles/`

Directory for user-generated output files produced by scripts from `Tools/` (Excel reports, CSV exports, logs, etc.). Scripts should write their default output here. This directory is not imported back into TIA Portal and is excluded from version control artifacts.

---

## Siemens TIA Portal Export Files (Openness XML)

The XSD schemas in `.github/Schemas/` define the valid structure for all TIA Portal Openness XML files. **Always** follow these schemas when creating or modifying `.xml` block files.

---

### S7 Declaration and Resource Files (.s7dcl + .s7res)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cmariusz/TiaImportExport.VSExt](https://github.com/cmariusz/TiaImportExport.VSExt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
