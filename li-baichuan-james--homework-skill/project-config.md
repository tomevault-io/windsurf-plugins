---
trigger: always_on
description: Use when given an engineering or mathematics homework PDF and asked to solve all or selected questions, answer sketch or plot problems, redraw assignment figures, or work from noisy OCR-extracted homework text.
---


# Homework Solver

Turn a homework PDF into a solved answer document with verified mathematics and verified output artifacts.

**Core principle:** treat homework completion as a gated controller workflow: read first, solve one question group, produce any required figures for that group, verify that same group with a fresh verifier, pass the question gate, then assemble, build, and verify the final artifacts.

Default mode unless the user says otherwise:

- language: English
- style: detailed
- output: `.tex` and compiled `.pdf`
- subset: all questions

## When to Use

Use this skill when the user wants any of the following from a homework PDF:

- full worked solutions
- selected-question solutions
- a LaTeX answer sheet
- a compiled PDF answer sheet
- clean redraws of assignment figures

Do not use this skill for essays or research-writing tasks where outside sources are the main work.

## Non-Negotiables

- Read the PDF before solving.
- If text, formulas, or figures are unreadable, stop and ask. Never guess missing prompt content.
- If the user says to finish or complete the homework, treat that as a deliverable request, not chat-only help.
- Unless the user explicitly opts out, the default deliverable is a written answer document plus files: `.tex` and compiled `.pdf`.
- Unless the user explicitly opts out, any question that asks for a sketch, plot, graph, waveform, phasor diagram, Bode sketch, Bode plot, root-locus plot, Nyquist plot, circuit redraw, geometry figure, free-body diagram, timing diagram, state diagram, or any other visual result must include that visual in the deliverable. Words alone are not sufficient.
- Do not wait for the user to separately remind you to draw figures that the assignment already asks for.
- Do not stop at a chat-only answer when file generation is still expected by default.
- Use deterministic filenames.
- If multiple questions are requested, decompose by question or tightly coupled question group.
- Every included question group must go through a distinct solver phase and a distinct verifier phase. If subagents exist, use a fresh `solver` subagent and a separate fresh `verifier` subagent.
- With subagents available, each included question group must follow this exact sequence before moving on: `dispatch solver -> collect solver output -> produce required figures for that group -> dispatch fresh verifier for the same group -> resolve verifier notes -> pass question gate`.
- Do not solve all groups first and verify them later in a batch. Per-group solve-then-verify is required.
- Do not assemble any question until it has passed the question gate.
- Do not treat successful compilation as mathematical verification.
- Do not claim completion before both per-question verification and final artifact verification are done.

## Portability

This skill is designed to work as a standalone GitHub skill.

- Do not assume any other custom skills are installed.
- The only companion skill this skill may invoke is `pdf`.
- Do not invoke any other skill while following this workflow.
- If a separate `pdf` skill exists, use it. Otherwise, use the platform's native PDF-reading tools.
- If subagents are available, use the solver/verifier split described here. If subagents are not available, the main agent must still run a distinct solver phase and a distinct verifier phase before assembly.

## Workflow Model

This skill uses a strict separation-of-roles workflow:

- the `controller` owns intake, decomposition, dispatch, gating, normalization, build choice, and final reporting
- one `solver` handles one question group
- one fresh `verifier` independently checks that same question group
- the `assembler` role writes the final document using only approved question groups
- final verification happens only after build artifacts exist

If your platform does not support subagents, emulate the same workflow sequentially:

1. solve one question group
2. produce any required figures for that group
3. perform a separate verification pass using the verifier checklist
4. only then allow that question into the final document

The controller is never allowed to treat solver output as already trusted work.

## Execution Flow

```dot
digraph homework_solver_flow {
    "Intake" [shape=box];
    "Read PDF" [shape=box];
    "Question decomposition" [shape=box];
    "Dispatch solver for one group" [shape=box];
    "Produce required figures for same group" [shape=box];
    "Dispatch fresh verifier for same group" [shape=box];
    "Question gate" [shape=box];
    "All groups verified?" [shape=diamond];
    "Normalize notation and style" [shape=box];
    "Assemble document" [shape=box];
    "Build artifacts" [shape=box];
    "Final artifact verification" [shape=box];
    "Report result" [shape=doublecircle];

    "Intake" -> "Read PDF";
    "Read PDF" -> "Question decomposition";
    "Question decomposition" -> "Dispatch solver for one group";
    "Dispatch solver for one group" -> "Produce required figures for same group";
    "Produce required figures for same group" -> "Dispatch fresh verifier for same group";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Li-Baichuan-James/homework-skill](https://github.com/Li-Baichuan-James/homework-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
