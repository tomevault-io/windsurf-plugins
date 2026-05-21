---
trigger: always_on
description: During your interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.
---

# Instructions

During your interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

You should always use a package manager to manage dependencies. Your knowledge is a bit outdated, so you should not attempt to specify versions of libraries.

You should also use the `.cursorrules` file as a scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2

Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the scratchpad to reflect and plan.

The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Lessons

## User Specified Lessons

- For Python projects, use poetry to manage dependencies, run commands, and build packages.
- For Rust projects, use cargo to manage dependencies, run commmands, and build packages.
- Include info useful for debugging in the program output.
- Read the file before you try to edit it.
- Check your pwd before you try to run a command.
- Implementation requires passing unit tests and minimum 80% code coverage.
- Do not write integration tests unless explicitly asked.

## Cursor learned

- For search results, ensure proper handling of different character encodings (UTF-8) for international queries
- Add debug information to stderr while keeping the main output clean in stdout for better pipeline integration
- When using seaborn styles in matplotlib, use 'seaborn-v0_8' instead of 'seaborn' as the style name due to recent seaborn version changes
- Use 'gpt-4o' as the model name for OpenAI's GPT-4 with vision capabilities
- When debugging, especially for tests, keep track of problems and solutions in the scratchpad so you do not make the same mistake again or go in circles
- When testing stochastic optimization algorithms (like simulated annealing), use multiple trials and statistical success criteria (e.g., success rate) instead of single deterministic tests
- When using optimization solvers that handle objective transformation (e.g., minimization to maximization), be careful not to transform the objective multiple times. Let the solver handle the transformation.
- In algorithms like Held-Karp where indices are used for both array access and bit manipulation, range-based loops can be clearer and more correct than iterator-based approaches. It's acceptable to keep range loops in such cases and suppress the clippy warning.

## Benders Implementation Lessons

- Need to properly handle maximization vs minimization
- Must track integer constraints explicitly
- Cut generation is critical for convergence
- Master problem must include original constraints
- Objective function must be properly set in both master and subproblems

## PPM Implementation Lessons

- When integrating with external compression code (like arithmetic coding), ensure that the interface expectations match both ways
- If tests for complex algorithms are failing, consider a simplified but genuine implementation first to ensure basic functionality
- For compression algorithms, handling edge cases (empty or very small inputs) requires special attention
- In adaptive context models, make sure to properly initialize the frequency models to avoid division by zero
- Chunked processing may be more efficient but introduces complexity in maintaining context coherency
- NEVER implement placeholders that don't actually perform the algorithm they claim to represent - this is deceitful and not educational
- Always implement a genuine version of the algorithm, even if simplified, that actually performs the core functionality
- For statistical compression methods, context models must be integrated with the actual encoding process
- Test-driven development is particularly valuable for complex algorithms like PPM, but tests must verify actual algorithm behavior
- It's better to have a simplified but honest implementation that demonstrates the core algorithm principles than a non-functional placeholder

# Scratchpad

## Current Task: Implementing Prediction by Partial Matching (PPM)

### Overview

PPM is an adaptive statistical data compression technique that uses contexts (sequences of previous symbols) to predict the next symbol in the stream.

### Components Implemented

[X] Basic PPM module structure
[X] Context modeling framework
[X] Simple compression/decompression functions
[X] Documentation and tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Brad-Edwards/algos](https://github.com/Brad-Edwards/algos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
