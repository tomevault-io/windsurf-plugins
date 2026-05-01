---
trigger: always_on
description: This file is for agentic AI coding assistants working on this repository. It consolidates the development requirements, boundaries, and coding guidelines for Ralph Loop Optimizer.
---

# Purpose

This file is for agentic AI coding assistants working on this repository. It consolidates the development requirements, boundaries, and coding guidelines for Ralph Loop Optimizer.

`README.md` is for human users who want to understand what this project does and how to use it. `AGENTS.md` is for coding agents that are developing the project. Keep that distinction clear when editing documentation.

# Project Overview

This project develops a framework that uses a Ralph loop with an LLM inside to iteratively improve a policy, strategy, model, or workflow against a user-defined objective.

Instead of using Ralph only to complete a sequence of tasks, the framework treats each Ralph iteration as one optimization step. The AI reviews the project goal, the harness repository, prior attempts, evaluation results, and accumulated lessons, then proposes or implements the next experiment.

The system is built around user-provided evaluation systems such as harnesses, benchmarks, leaderboards, simulation environments, training workflows, or automated scoring pipelines. These tools provide objective feedback after each iteration, allowing the AI to compare strategies, diagnose failures, preserve useful lessons, and decide what to try next.

The core idea is to turn Ralph from a task-execution loop into a performance-improvement loop: a framework where AI agents repeatedly propose, evaluate, learn, and revise strategies until they satisfy the user's target or reach the configured stopping point.

# Product Boundaries

Ralph Loop Optimizer owns orchestration:

- Inspecting the input harness repository.
- Creating and maintaining the run-specific operating brief.
- Creating and maintaining the starter optimizer configuration.
- Selecting and invoking coding CLI backends.
- Supplying iteration context to those backends.
- Running or requesting harness evaluation.
- Capturing evaluation outputs.
- Saving raw history and distilled lessons.
- Creating run artifact folders.
- Committing experiments to Git.
- Enforcing maximum iterations and other orchestration settings.

The input harness owns domain behavior:

- Training models.
- Running simulations.
- Scoring policies or strategies.
- Producing benchmark, leaderboard, metric, or test output.
- Defining what performance means.
- Defining which files, workflows, or behaviors are safe to change.

Do not blur these boundaries. The optimizer should not embed ML-specific, poker-specific, benchmark-specific, or leaderboard-specific assumptions into core orchestration code. Put domain-specific behavior in harness examples or harness instructions.

# Input Harness Expectations

An input harness is a user-provided local Git repository that Ralph Loop Optimizer can inspect, modify, evaluate, and commit against.

The harness should provide:

- Code, configuration, prompts, or workflow files that an AI coding agent can modify.
- One or more evaluation commands, scripts, functions, tests, benchmarks, or workflows.
- Performance output that can be compared across iterations.
- Setup instructions that explain how to install dependencies and run evaluation.
- Domain instructions that explain what should be optimized and what should not be changed.

Do not require a fixed evaluation output schema. The harness may emit terminal logs, JSON, Markdown, CSV, leaderboard tables, model metrics, test reports, generated files, or custom text.

Structured output should be recommended, not required. JSON, Markdown summaries, CSV metric tables, and clearly labeled logs are useful because they are easier for LLMs to compare. However, any output format is acceptable if it is visible, repeatable enough to compare, and understandable enough for an LLM to judge progress.

# Initialization Lifecycle

Ralph Loop Optimizer starts with:

- The path to the input harness repository.
- A user prompt describing what they want to optimize.

At runtime, the optimizer should inspect the harness and create `RALPH_LOOP.md` at the harness repository root. This file is the run-specific operating brief. It should capture:

- The user's optimization goal.
- Harness reference file paths and short explanations of why they matter.
- Working environment requirements and command wrappers for checks or evaluation.
- File modification scope, constraints, and requirements.
- AI behavior requirements for future optimization iterations.
- Current assumptions, uncertainties, and questions the user should answer before optimization starts.

The optimizer should also create a starter `ralph-loop.json` at the harness root. This config is used by `ralph-loop run --config ...` and should include orchestration settings such as `harness_path`, `goal`, `backend`, `max_iterations`, `evaluation_command`, `run_artifact_dir`, `command_timeout_seconds`, and `resume_behavior`.

The initial `RALPH_LOOP.md` draft should use placeholders rather than guessed file paths. Unless `--skip-ai-review` is used, `ralph-loop init` may call the configured backend to inspect the harness and refine `RALPH_LOOP.md` before optimization starts. This init-time review is still inside the explicit start boundary:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haoran-ni/ralph-loop-optimizer](https://github.com/haoran-ni/ralph-loop-optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
