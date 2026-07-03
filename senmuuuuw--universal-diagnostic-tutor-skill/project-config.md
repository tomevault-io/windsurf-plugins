---
trigger: always_on
description: Guidance for future Codex maintenance work on this repository.
---

# AGENTS.md

Guidance for future Codex maintenance work on this repository.

## Project Intent

This repository contains a reusable Codex Agent Skill for universal,
diagnosis-first tutoring across all subjects.

Future changes must preserve the core identity of the skill:

- It is universal across subjects.
- It is diagnosis-first, not answer-first.
- It supports mastery, not just task completion.
- It teaches from the learner's likely starting point.

## Maintenance Rules

- Do not narrow this skill into a SAT-only, math-only, coding-only, or
  single-subject tutor.
- Keep STEM / science / AI-CS as the clearest public positioning while
  preserving universal-capable diagnosis-first tutoring.
- Do not describe the Skill primarily as a generic all-subject assistant.
- Preserve the diagnosis-first workflow in `SKILL.md`.
- Keep `SKILL.md` concise enough to load as a skill body.
- Keep `SKILL.md` as a router, not a duplicate of every protocol.
- Put detailed subject guidance in `skills/universal-diagnostic-tutor/references/`.
- Prefer routing clarity over protocol sprawl.
- Update examples when behavior or answer formats change.
- Update `CHANGELOG.md` for user-visible changes.
- Add eval cases for repeated failures before broadening the Skill.
- Keep examples short, clear, and representative across subject areas.
- Preserve adaptive teaching behavior: diagnose the learner's current gap,
  teach the next useful chunk, check understanding, and adapt across turns.
- Do not solve by explaining more if a smaller intervention would work.
- Prefer the next best teaching step over a full lecture.
- Treat learner responses as mastery signals, not only right/wrong outcomes.
- Compress explanations when prerequisites are already known.
- Map mistake type to intervention type: notation, concept, method selection,
  setup, proof, calculation, transfer, overgeneralization, or memorized
  procedure.
- Keep README focused; the learning-efficiency loop is a selling point, not a
  complex technical diagram.
- Do not make README too long; link dedicated evaluation, rubric, taxonomy, and
  improvement docs instead of duplicating them.
- Preserve teacher-like pacing: teach one useful chunk, check, then continue.
- Preserve learner mode calibration: Auto, Zero-Base, Standard, and Advanced
  should be chosen from learner evidence, not treated as rigid labels.
- Do not assume a beginner knows notation, symbols, object types, or
  prerequisites.
- In Zero-Base Mode, explain symbols and objects before proof, theorem use, or
  full solution.
- After a check question, stop and wait for the learner instead of continuing
  to the next proof step, subproblem, theorem idea, or final result.
- Do not slow down advanced learners unnecessarily; if they ask for rigor,
  proof, derivation, edge cases, or concise explanation, keep the answer
  efficient unless a gap appears.
- Allow mode switching based on learner response. Step down for notation or
  prerequisite gaps; step up when the learner shows readiness.
- Do not expose mode labels too often if natural phrasing is better.
- Do not let the tutor solve too much at once.
- If the learner requests no direct answer, preserve at least one meaningful
  step for the learner unless they are stuck or ask for the final answer.
- Prefer one problem or subproblem at a time unless the user explicitly asks
  for a complete multi-question solution.
- When the learner says they do not understand, do not repeat the same
  explanation; step down, change representation, and rebuild from the missing
  prerequisite.
- Do not let source packs replace teaching. Sources may support explanations,
  practice, and study paths, but the tutor still needs to diagnose and teach.
- Do not assume users will upload materials. When web/search access is
  available and resources would improve teaching, verification, practice
  design, or exam-pattern analysis, the tutor may actively search for
  authoritative learning resources.
- When resources are used, integrate them into teaching instead of dumping
  links.
- Do not fabricate resources, citations, exams, course pages, authors,
  institutions, page numbers, or links.
- Make the Skill's difference from generic AI visible in examples and docs:
  diagnosis, pacing, stop points, resource discovery, resource orchestration,
  mistake analysis, checks, difficulty adjustment, and transfer.
- When adding adaptive examples, include diagnosis, knowledge gap type,
  adaptive response, and a practice or understanding check.
- Do not make every answer a rigid template. Preserve natural teaching style
  and use visible labels only when they help the learner.
- V0.8 focuses on STEM / AI-CS teaching calibration while preserving universal
  scope.
- Do not make adaptive STEM teaching overly robotic or over-labeled.
- Normal tutoring answers should not expose internal Skill names, version
  numbers, repository files, protocol names, acceptance reports, or
  implementation details unless the user explicitly asks about the project.
- Do not expose internal gap labels such as "notation gap" or "concept gap"
  unless naming the label genuinely helps the learner.
- Prefer natural teacher language such as "The part that may be causing trouble
  is..." over diagnostic labels.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SenmuuuuW/universal-diagnostic-tutor-skill](https://github.com/SenmuuuuW/universal-diagnostic-tutor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
