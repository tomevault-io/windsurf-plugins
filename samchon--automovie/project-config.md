---
trigger: always_on
description: `automovie` lets an LLM perform a fixed asset through function calling and a deterministic engine render it, as the cheap, reproducible alternative to diffusion video. The [project skill](.agents/skills/project/SKILL.md) owns the product contract.
---

# AGENTS.md

`automovie` lets an LLM perform a fixed asset through function calling and a deterministic engine render it, as the cheap, reproducible alternative to diffusion video. The [project skill](.agents/skills/project/SKILL.md) owns the product contract.

## Attitude

Follow the literal request; it is the contract, not a hint at what the user "really" wants.

- **The user outranks a skill.** A direct instruction outranks this file and every skill, and a standing instruction recorded here keeps that authority until the user changes it. Infer no override from a terse request or a deadline.
- **Name the instruction that stopped you.** Before a skill makes you pause, refuse or narrow the request, quote the exact file and sentence. Record an override in the pull-request chronology or the `.wiki/` worklog, not in the durable rule.
- **Scope is the user's to widen.** Expand or reinterpret the task only on an explicit hand-off ("you decide"), and report an unrelated defect as a follow-up unless the requested behavior cannot work without fixing it. Inside the goal, act with full initiative.
- **Choose the principled course.** Decide from correctness, evidence and durable consequence. Size, difficulty and blast radius change how much investigation a decision needs, never the standard it must meet.
- **Evidence precedes correction.** Treat a report, a claim that something is wrong or missing, and your own recall of anything outside this repository as hypotheses. Verify the real code, output and history first.
- **Default over ask.** Pick the sensible default and say what you chose. Ask only about forks the user alone can settle, and take the reversible step a request implies. A problem report or a question calls for your assessment.
- **Match the user's language.** Answer in English or Korean as the user writes, and switch when they switch.
- **Finish the turn's work.** A message with no tool call ends the turn, so keep the request's parts in a task list you update, and treat an ending with items open and no stated blocker as unfinished.
  - End no turn with a summary that announces the next step without taking it, an offer to continue unless the user objects, a list of decisions when none blocks the rest of the work, or a decision that this is a good place to report.
  - Put status notes in the message with your next tool call.
  - Stop only where nothing can move without the user or a skill withholds the action. Session length is never a reason, and confirmation before a destructive or outward-facing action still applies.
- **Background work never ends the turn.** While a build, test run or CI job runs, keep reviewing, developing or researching the next item, and check the job with a targeted command instead of sleeping.
- **Collect every symptom before correcting.** When a check or review reports failures, read all of them, find the shared cause, and fix the class in one pass.
- **Recheck every fifteen minutes.** In any work, pause and ask whether you still serve the literal request, whether the approach is principled under the [contracts skill](.agents/skills/contracts/SKILL.md)'s common chapters or has become a chain of workarounds, and whether what you learned changes the plan. Correct course at once, record the correction in the pull-request chronology or the `.wiki/` worklog, and continue.
- **Record every user instruction** in the `.wiki/` worklog at once, and keep a superseded one beside its replacement ([`.wiki/` document](.agents/skills/documentation/wiki.md)).
- **Ship each topic as its own PR** and never commit to `master` directly. The [pull-request skill](.agents/skills/pull-request/SKILL.md) owns the flow and the merge conditions.

## Skills

Read a skill when its topic applies. Each skill links its own conditional topic documents.

### [Project](.agents/skills/project/SKILL.md)

Product contract, decided exclusions, workspace layout, commands. Read before judging whether a capability is in scope or choosing a command.

### [Development](.agents/skills/development/SKILL.md)

Source and test rules, the per-change 100% coverage obligation, validation. Read before writing or changing code.

### [Contracts](.agents/skills/contracts/SKILL.md)

Layered implementation checklists that source declarations answer with `@evidence` tags. Read before implementing or reviewing maintained source.

### [Scaffold Authoring](.agents/skills/scaffold/SKILL.md)

The self-contained harness every generated project inherits (`packages/template`). Read before editing it, and before interpreting, authoring or reviewing production content anywhere, fixtures and sandboxes here included.

### [Documentation](.agents/skills/documentation/SKILL.md)

`.wiki/`, package READMEs, JSDoc, and the writing rules for agent instructions. Read before writing docs or instructions.

### [Evidence Graph](.agents/skills/evidence-graph/SKILL.md)

The requirement, specification and public-source triangle, and the scaffold contract corpus. Read before changing those sources, public-export evidence JSDoc or repository `@ttsc/evidence` configuration.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samchon/automovie](https://github.com/samchon/automovie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
