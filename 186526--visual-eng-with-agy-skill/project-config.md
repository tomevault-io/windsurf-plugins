---
trigger: always_on
description: >-
---


# Visual Engineering With agy

Route complex styling-focused visual engineering work through a low-cost general agent that runs `agy` with Gemini 3.1 Pro, while keeping the lead agent responsible for scope, review, and verification.

## agy and Antigravity Self-Ignore Rule

This skill is only for OpenCode/Sisyphus/Oh My OpenAgent routing decisions before launching `agy`. If this file is loaded inside `agy`, Antigravity CLI, or an `agy`-spawned agent, ignore the entire skill and continue with the actual user prompt. Do not apply this skill recursively, do not launch another `agy` process because of this skill, and do not treat `MUST LOAD` or routing instructions in this file as applicable inside `agy` itself.

## Mandatory Loading

This skill is mandatory for complex styling-focused visual engineering work that matches the conditions below. When the task qualifies, the orchestrator should load `visual-eng-with-agy` before delegating implementation.

Critical routing rule: do not route qualifying work directly to OMO's `visual-engineering` category first. That bypasses the `agy` workflow by selecting the OMO visual engineering model before this skill can run. For qualifying work, delegate to a low-cost general-purpose agent and load this skill explicitly with `load_skills=["visual-eng-with-agy"]`.

The mandatory marker is intentionally narrow:

- Mandatory for complex visual/styling work that needs design judgment, multi-surface consistency, screenshot-driven QA, or broad visual decomposition.
- Not mandatory for frontend logic, state, data flow, API wiring, event handling, tests, or behavior changes.
- Not mandatory for simple styling tweaks that a regular general agent can complete directly.

If the harness supports explicit skill loading, pass `load_skills=["visual-eng-with-agy"]` for qualifying tasks and use a low/general implementation category rather than `visual-engineering`.

## Correct Task Routing Examples

Wrong: this loads the skill text but still routes execution through OMO's visual-engineering model, so `agy` may never run.

```text
task(category="visual-engineering", load_skills=["visual-eng-with-agy"], run_in_background=false, prompt="...")
```

Correct for complex styling work: use a low/general category and load this skill so the wrapper agent follows the `agy` workflow.

```text
task(category="unspecified-low", load_skills=["visual-eng-with-agy"], run_in_background=false, prompt="Launch agy with Gemini 3.1 Pro per the visual-eng-with-agy skill. First decompose the visual task, then execute one atomic visual change per agy run.")
```

Use `unspecified-high` instead of `unspecified-low` only when the wrapper work itself needs more reasoning or broader project context. Do not use `visual-engineering` as the first category for this skill; reserve OMO visual-engineering for the fallback chain when `agy` is unavailable.

Correct for complex visual review or static audit tasks: still require the wrapper agent to launch `agy` first. Review-only does not mean agy-optional.

```text
task(category="unspecified-low", load_skills=["visual-eng-with-agy"], run_in_background=false, prompt="Review this visual change without editing files. You MUST launch agy with Gemini 3.1 Pro first and use its output as the review basis. If agy is unavailable, report why before using any fallback.")
```

## Lead Route Accountability

The lead agent must always state the intended route before delegating a qualifying frontend design task:

- If using `agy`, say that the subagent must launch `agy` first and include the exact `agy` requirement in the subagent prompt.
- If not using `agy`, explain why this route does not use `agy` before delegating: simple styling tweak, frontend logic, `agy` unavailable, unauthenticated, timeout, or explicit user override.
- The subagent prompt must require the subagent to report whether it launched `agy` and, if not, the exact reason.
- A subagent response that omits whether `agy` was launched is incomplete and must be followed up before accepting the result.
- Review-only and no-edit tasks still need this route statement.

## When To Use

Use this skill when all of these are true:

- The task is primarily about styling, CSS, visual layout, spacing, typography, color, animation, screenshots, responsive presentation, or browser-visible polish.
- The visual change is complex enough to benefit from a dedicated visual pass: multi-component styling, design-system alignment, responsive layout work, animation choreography, screenshot-driven polish, or a broad visual redesign.
- The user wants Oh My OpenAgent/OhMyOpenCode orchestration rather than direct implementation by the lead agent.
- A low-cost general-purpose agent is acceptable for execution, with Gemini 3.1 Pro providing the visual engineering reasoning inside `agy`.

Do not use this skill for backend-only work, non-visual refactors, pure documentation, frontend logic, state management, API wiring, data loading, validation, event handling, routing behavior, simple styling tweaks, one-off CSS fixes, minor spacing/color changes, or visual tasks that require direct specialist execution instead of `agy`.

## Complexity Threshold

Prefer a regular general agent for simple styling work. Use Visual Engineering With agy only when the visual task is meaningfully complex.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [186526/visual-eng-with-agy.skill](https://github.com/186526/visual-eng-with-agy.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
