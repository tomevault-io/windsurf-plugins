---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a conventions repository that defines project-wide standards for Claude Code sessions on **Python** projects. The primary artifact is `CLAUDE.md`.

## Environment

This project runs inside a **Docker container** with [Claude Code](https://claude.ai/code) as the primary development tool.

| Item              | Detail                                         |
|-------------------|-------------------------------------------------|
| Runtime           | Docker container (`--privileged`)               |
| OS                | Ubuntu 24.04 (Noble)                            |
| Dev tool          | Claude Code (CLI / VS Code extension)           |

## 1. Rule Priority

Project-level `CLAUDE.md` files take precedence over this global ruleset. Specific rules beat general ones. When a conflict arises, the more-specific context wins.

---

## 2. MIT Code Convention

All code follows the [MIT CommLab Coding and Comment Style](https://mitcommlab.mit.edu/broad/commkit/coding-and-comment-style/).

### Naming

- **Variables and classes** are nouns; **functions and methods** are verbs.
- Names must be pronounceable and straightforward.
- Name length is proportional to scope: short for local, descriptive for broad.
- Avoid abbreviations unless self-explanatory. If unavoidable, define them in a comment.
- Python conventions:

| Element  | Style        | Example            |
|----------|--------------|--------------------|
| Variable | `lower_case` | `joint_angle`      |
| Function | `lower_case` | `send_action`      |
| Class    | `CamelCase`  | `RobotState`       |
| Constant | `lower_case` | `settle_mid_ms`    |
| Module   | `lowercase`  | `robot_state`      |

### Structure

- **80-column limit** for all new code.
- One statement per line.
- Indent with **4 spaces** (never tabs).
- Place operators on the **left side** of continuation lines so the reader can see at a glance that a line continues.
- Group related items visually with alignment.

### Spacing

- One space after commas, none before: `foo(a, b, c)`.
- One space on each side of `=`, `==`, `<`, `>`, etc.
- Be consistent with arithmetic operators within a file.

### Comments

- Use **complete sentences**.
- Only comment for **context** or **non-obvious choices**. Never restate what the code already says.
- Outdated comments are worse than none. Keep them current or delete them.
- TODO format:
  ```python
  # TODO: (@owner) Implement 2-step predictor-corrector
  # for stability -- Adams-Bashforth causes shocks.
  ```

### Language

- All code comments, docstrings, commit messages, documentation files (including README), **GitHub issues, and pull requests** must be written in **English**.

### Documentation

- All public functions and classes must have **docstrings** following [PEP 257](https://peps.python.org/pep-0257/) (Google style recommended).
- A docstring states **what** and **why**, not **how**.
- Include `Args:`, `Returns:`, and `Raises:` sections when applicable.

Example:
```python
def send_action(joint_pos: list[float]) -> dict:
    """Stream a servo joint command to the robot.

    The command is non-blocking and returns once the controller
    acknowledges receipt -- it does not wait for motion to finish.

    Args:
        joint_pos: Six target joint angles in degrees, ordered
            from base (J1) to flange (J6).

    Returns:
        Controller acknowledgement payload with keys
        ``"errcode"`` and ``"timestamp"``.

    Raises:
        ConnectionError: If the controller socket has dropped.
        ValueError: If ``joint_pos`` is not exactly six values.
    """
```

---

## 3. Debug File Management

All debug, exploratory, and throwaway test scripts must be saved in `claude_test/`, **not** in `tests/`.

### Rules

| Location        | What goes there                                      |
|-----------------|------------------------------------------------------|
| `tests/`        | Production-quality tests that are part of CI/CD.     |
| `claude_test/`  | Debug scripts, one-off experiments, diagnostic code. |

### When writing debug code

1. Create the file directly in `claude_test/` (e.g., `claude_test/debug_servo_timing.c`).
2. Add a one-line comment at the top explaining the purpose.
3. If the debug script leads to a real fix, move the relevant parts into a proper test under `tests/` and delete or archive the debug version.

### README

`claude_test/README.md` is the index. When adding a new debug file, add a row to the table in that README describing what the file does and what was learned.

---

## 4. Task Management

> **MANDATORY**: This workflow applies to **every task without exception**, regardless of size or complexity. No task may begin without writing `ToDo.md` and creating a GitHub issue via `gh`. Skipping any step is not allowed.

### Rules

1. **Write ToDo.md**: For every task requested by the user, create a `ToDo.md` file and confirm the contents with the user before starting work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coport-uni/HotplateController](https://github.com/coport-uni/HotplateController) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
