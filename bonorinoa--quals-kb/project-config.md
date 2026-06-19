---
trigger: always_on
description: Build rigorous, self-contained, exam-facing PhD qualifying-exam lecture-note packets, playbooks, exercise banks, and problem-solving labs from professor notes, problem sets, past exams, and supplementary theory sources.
---


# PhD Qual Lecture Notes Builder

## Purpose

Use this skill to create, revise, audit, or plan qualifying-exam study materials that teach a student how to solve exam problems under time pressure. The output should not merely summarize lecture notes. It should convert source material into a problem-solving system: definitions, canonical models, derivation templates, closed-form methods, common traps, targeted exercises, and past-exam mappings.

The skill is field-general. It can be used for macroeconomics, microeconomics, econometrics, math economics, or any PhD qualifier area. Domain-specific details should be supplied in a separate manifest, such as `MACRO_MANIFEST.md` or `MICRO_MANIFEST.md`.

## Use Cases

Use this skill when the user asks for any of the following:

- A full lecture-note packet for one qualifier module.
- A module-specific exercise bank.
- A worked-problem lab.
- A methodology or playbook document.
- A revision or quality audit of an existing module packet.
- A source inventory and module plan.
- A TeX/PDF study packet.
- A harmonization pass across multiple module packets.

Do not use this skill for quick one-off answers unless the user explicitly wants the answer to follow the project methodology.

## Required Inputs

Before drafting, identify or request the following inputs. If some inputs are missing but the task is still feasible, proceed with explicit assumptions rather than blocking.

1. Field/course, for example: `ASU Macro Theory Qual`.
2. Module name, for example: `OLG Model`.
3. Source folder or uploaded files.
4. Past-exam folder or past-exam files.
5. Relevant problem sets or homeworks.
6. Desired output type: outline, source inventory, TeX, PDF, exercises, answer sketches, full packet, or audit report.
7. Desired depth: compact, standard, exhaustive, or problem-solving lab.
8. Target weakness, if any, for example: `closed-form derivations` or `RCE definitions`.
9. Style constraints, including notation preferences and whether the output should be professor-specific or exam-specific.

If a domain manifest is present, read it before drafting and follow its module definitions, source hierarchy, notation conventions, and output standards.

## Source Hierarchy

When sources conflict or differ in emphasis, use this hierarchy:

1. Past qualifying exams determine what is exam-relevant.
2. Current problem sets determine current-course emphasis.
3. Professor lecture notes and slides determine notation, terminology, and course framing.
4. Homeworks determine practice style and expected derivation depth.
5. Supplementary textbooks, papers, or external sources fill gaps only when needed.

Do not let professor notes crowd out the past-exam pattern. The packet must teach the student how to solve qualifier problems, not simply reproduce a course lecture.

## Mandatory Workflow

### Step 1. Source Inventory

Before drafting substantive notes, inventory the accessible sources.

For each source, record:

- file name or URL/path;
- source type: past exam, problem set, lecture note, homework, slide deck, textbook, paper, or prior student note;
- topics covered;
- mathematical tools used;
- exam relevance;
- any extraction or rendering issues.

If a source is inaccessible, say so plainly and continue with available sources.

### Step 2. Pattern Analysis

Identify patterns before writing notes.

For the assigned module, produce:

- core problem archetypes;
- recurring wording and signature instructions;
- recurring state variables;
- standard constraints;
- standard FOCs, envelope conditions, Euler equations, market-clearing conditions, or cutoff equations;
- closed-form opportunities;
- common mistakes and traps;
- past-exam mappings;
- links to current problem sets or homeworks.

### Step 3. Module Blueprint

Create a blueprint before writing the full packet.

The blueprint must include:

- module objective;
- prerequisite concepts;
- canonical models;
- exam variations;
- closed-form catalog;
- exercise plan;
- estimated study use, for example: `one 4-hour block` or `two 3-hour blocks`.

### Step 4. Draft the Packet

Use the standard packet structure below. Adapt only when the user requests a methodology playbook or a problem-solving lab.

### Step 5. Add Exercises and Answer Sketches

Every teaching packet must include targeted practice. Exercises should be explicitly tied to the module's exam archetypes.

### Step 6. Quality Audit

Before finalizing, run the quality audit below and either fix issues or disclose remaining limitations.

### Step 7. Output Files

If the user asks for files, produce clean, reusable files. If producing LaTeX, provide an Overleaf-ready `.tex` source. If producing PDF, compile it and spot-check readability. If compilation or rendering is unreliable, provide the `.tex` source and explain the limitation.

## Standard Packet Structure

Every normal lecture-note packet should use this structure unless the user requests otherwise.

### 1. Title Page and Orientation

Include:

- module title;
- course/qualifier;
- purpose of the packet;
- how to use it;
- expected study block length;
- source base.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bonorinoa/Quals-KB](https://github.com/Bonorinoa/Quals-KB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
