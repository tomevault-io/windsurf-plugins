---
trigger: always_on
description: > **This document is the source of truth for all agent behavior in this repository.**
---

# 🤖 Agent Execution Protocol

> **This document is the source of truth for all agent behavior in this repository.**
> Tasks are not considered complete unless every criterion in this protocol is satisfied.
> No exceptions.

---

## Table of Contents

1. [Core Principles](#1-core-principles)
2. [Planning & Verifiable States](#2-planning--verifiable-states)
3. [Code Quality Standards](#3-code-quality-standards)
4. [Progress Tracking](#4-progress-tracking)
5. [Mandatory Validation](#5-mandatory-validation)
6. [Completion Criteria](#6-completion-criteria)
7. [Quick Reference Checklist](#7-quick-reference-checklist)

---

## 1. Core Principles

Every agent operating in this repository must embody the following values:

| Principle | Description |
|-----------|-------------|
| **Correctness over speed** | A slow, correct solution is always preferred over a fast, broken one. |
| **Verifiability** | Every completed step must have a concrete, checkable proof of completion. |
| **Transparency** | Progress and blockers must be visible in the checklist at all times. |
| **Clean code** | Code must be readable, documented, and maintainable by the next person who touches it. |

---

## 2. Planning & Verifiable States

**Before writing a single line of code**, the agent must produce a written plan.

### 2.1 What a Good Plan Looks Like

Each plan must contain:

- A **clear goal statement** — one sentence describing what done looks like.
- A **numbered list of steps** — granular enough that each step maps to a single concern.
- A **verifiable state** per step — an objective, testable condition that proves the step is complete.

### 2.2 Verifiable State Examples

| Step | ❌ Vague State | ✅ Verifiable State |
|------|---------------|---------------------|
| Add a function | "Function exists" | "Function returns expected output for all defined test cases" |
| Refactor module | "Code looks cleaner" | "`uv run mypy .` reports zero errors on the module" |
| Fix a bug | "Bug seems fixed" | "Previously failing test now passes; no regressions introduced" |
| Add an endpoint | "Endpoint added" | "Integration test hits the endpoint and receives HTTP 200 with correct schema" |

> **Rule:** If a verifiable state cannot be checked automatically or by inspection in under 60 seconds, make it more specific.

---

## 3. Code Quality Standards

All code produced or modified by an agent must meet these standards **before** marking any step complete.

### 3.1 Docstrings

Every public function, method, and class **must** have a docstring. Use the following format:

```python
def calculate_discount(price: float, rate: float) -> float:
    """Calculate the discounted price after applying a percentage rate.

    Args:
        price: The original price in the base currency unit.
        rate: The discount rate as a decimal between 0.0 and 1.0.

    Returns:
        The final price after the discount has been applied.

    Raises:
        ValueError: If rate is not between 0.0 and 1.0.

    Example:
        >>> calculate_discount(100.0, 0.2)
        80.0
    """
    if not 0.0 <= rate <= 1.0:
        raise ValueError(f"Rate must be between 0.0 and 1.0, got {rate}")
    return price * (1 - rate)
```

**Docstring checklist:**
- [ ] One-line summary (imperative mood: "Calculate...", not "Calculates...")
- [ ] `Args` section for every parameter (type + purpose)
- [ ] `Returns` section describing what is returned
- [ ] `Raises` section if any exceptions are raised
- [ ] `Example` block for non-trivial functions

### 3.2 Clean Code Rules

```
✅  DO                                  ❌  DON'T
─────────────────────────────────────────────────────────────
Use descriptive names                   Use single-letter variables (except loop counters)
Keep functions under 30 lines           Write monolithic functions
One responsibility per function         Mix I/O with business logic
Prefer explicit over implicit           Leave unexplained magic numbers/strings
Write self-documenting code             Rely on comments to explain bad names
Handle errors explicitly                Silence exceptions with bare `except:`
Use type hints everywhere               Leave function signatures untyped
```

### 3.3 Type Hints

All function signatures must include complete type hints:

```python
# ✅ Correct
def fetch_user(user_id: int, include_inactive: bool = False) -> User | None:
    ...

# ❌ Wrong — no type information
def fetch_user(user_id, include_inactive=False):
    ...
```

---

## 4. Progress Tracking

The agent maintains a live checklist throughout the task. This checklist is the single source of truth for what has been done and what remains.

### 4.1 Checklist Format

```markdown
## Task: <Goal Statement>

### Plan

- [ ] Step 1 — <description>
      Verifiable state: <how to confirm this is done>

- [ ] Step 2 — <description>
      Verifiable state: <how to confirm this is done>

- [x] Step 3 — <description> ✓ completed
      Verifiable state: all unit tests for this module pass

### Validation

- [ ] `uv run pytest` — all tests pass
- [ ] `uv run mypy .` — zero type errors
```

### 4.2 Checklist Rules

- Mark steps `[x]` **only** when their verifiable state is confirmed — not when the work feels done.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Horsmann/sharepoint-to-text](https://github.com/Horsmann/sharepoint-to-text) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
