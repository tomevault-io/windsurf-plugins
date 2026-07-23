---
trigger: always_on
description: This project (Gum) provides UI solutions for game developers using C#. It includes:
---

# Gum Repository Guidelines

## What Is This?

This project (Gum) provides UI solutions for game developers using C#. It includes:
* A Common library which can run in any C# environment for layout and some UI control logic
* Runtime libraries for various platforms including MonoGame, KNI, and FNA. Also SkiaSharp and raylib.
* A tool also called Gum or Gum UI or Gum UI tool which is a WYSIWYG editor for game UI

## Project Direction

High-level direction for Gum — vision, roadmap, open strategic questions, and decision records (ADRs) — lives in `Direction/` at the repo root. **For any discussion of Gum's high-level goals, roadmap, or strategic decisions, read `Direction/README.md` first**, then the specific file for the topic. This is the strategy layer (*what* and *why*); it is separate from the operational guidance in this file and in the skills/agents (*how*), and from the published user-facing docs in `docs/`.

## Agent Workflow

For every task, read the guidelines in the matching agent file under `.claude/agents/` and follow them yourself, in your own context, before proceeding — do not dispatch an actual subagent for this by default. Spinning up a subagent costs real tokens: a fresh agent has none of the context already built up in this conversation and has to independently re-read files, re-run builds, and re-explore the codebase to reconstruct it, on top of whatever the task itself costs. Reserve `Agent`-tool dispatch for two cases only: **(1)** the work is genuinely parallelizable (independent pieces that can run concurrently), or **(2)** the user explicitly asks for delegation/a subagent.

**Re-read the agent file at the start of each new task — not once per session.** Long sessions drift; reloading the discipline keeps it active. Inline coding without re-reading the agent file first is not an option, even for "small" follow-ups in the same conversation.

Also load any skill whose trigger matches the area you're working in — before reading code, designing a fix, or making changes. "I'm only investigating, not editing yet" is not a reason to skip; the skill exists to inform the investigation, not just the keystrokes. The only time it's acceptable to skip is for a trivial single-file lookup that won't influence any recommendation or change.

**This check re-runs per new file path, not once per task.** Matching skills against the files identified at task start (e.g. the source file a bug lives in) does not cover a different file touched later in the same task — a sample or test project pulled in only for manual verification, a config file edited in passing, a doc updated alongside. Each new file gets its own trigger-match against the skill list before it's edited, even when the task's "main" skill is already loaded.

Available agents:
- **coder** — Writing or modifying code and unit tests for new features or bugs
- **refactoring-specialist** — Refactoring and improving code structure
- **docs-writer** — Writing or updating documentation
- **product-manager** — Breaking down tasks and tracking progress
- **security-auditor** — Security reviews and vulnerability assessments

Select the agent that best matches the task at hand. For tasks that span multiple concerns (e.g., implement a feature and write tests), invoke the relevant agents in sequence.

**Reviewing changes before merge is not a dedicated agent.** Use the `/code-review` skill at **low or medium effort** (e.g. `Skill({skill: "code-review", args: "medium"})`) for routine pre-commit review — it covers correctness bugs *and* quality/refactoring cleanups in one pass, inline, no subagents. The coder writes its own unit tests; the `tdd` skill owns test discipline and the testability gate.

**Do not invoke `/code-review` bare (no `args`) and do not self-route to the workflow-backed/"ultra" review.** Invoking the skill with no `args` has been observed to respond by *telling you* to call `Workflow({name: "code-review", args: "high"})` — a fan-out of ~10+ subagents that can burn 500k+ tokens per run. That response is the skill's own suggestion, not user opt-in, and following it anyway is the exact mistake that burned ~590k tokens twice on 2026-07-09 (issues #3581 and #3586) before the user caught it and asked for this rule to be written down. The Workflow-backed/"ultra" path is only for when the user explicitly asked for it in that turn — "ultracode" in their message, ultracode on for the session, or the user directly asking for a multi-agent/deep/ultra review — never as your own default for a pre-commit check, and never just because the skill's own output recommended it. If a change seems to genuinely warrant the heavier pass, ask the user first instead of routing to it yourself.

## Improving Guidance Files Alongside Work


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vchelaru/Gum](https://github.com/vchelaru/Gum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
