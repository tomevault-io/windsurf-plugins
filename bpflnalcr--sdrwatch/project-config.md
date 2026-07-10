---
trigger: always_on
description: Keep guidance short and operational. Prefer **small diffs** over whole-file rewrites. Prioritize **runtime topology**, **baseline-first DB contracts**, **CLI↔service wiring**, **device locking**, and **tests**.
---

# SDRwatch — AI coding assistant guide (for Copilot/ChatGPT)

Keep guidance short and operational. Prefer **small diffs** over whole-file rewrites. Prioritize **runtime topology**, **baseline-first DB contracts**, **CLI↔service wiring**, **device locking**, and **tests**.

---

## 0) UNIX Programming Principles (Design Constraints)

This codebase follows the principles of UNIX programming. When making design or implementation decisions, prefer clarity, modularity, composability, and diagnosability over cleverness or premature optimization. Deviations are acceptable only when justified by measured requirements.

### Core Principles

1. **Modularity** — Design software as small, well-defined modules with single responsibilities and explicit interfaces. Avoid hidden coupling.

2. **Readability** — Optimize code for human understanding first. Use clear naming, straightforward control flow, and consistent structure. Avoid clever or opaque constructs.

3. **Composition** — Prefer building behavior by composing smaller components (pipelines, adapters, orchestrators) rather than monolithic implementations.

4. **Mechanism vs. Policy** — Separate capabilities (mechanisms) from decisions (policy). Do not hard-code policy where configuration or higher-level orchestration is appropriate.

5. **Simplicity** — Implement the simplest solution that correctly solves the current problem. Avoid unnecessary abstraction or speculative generality.

6. **Smallness** — Keep functions, modules, and services small enough to be fully understood, tested, and replaced. Split components that grow beyond clear conceptual bounds.

7. **Transparency** — Make program behavior observable and inspectable. Prefer explicit state, clear logging, and traceable data flow over implicit or "magical" behavior.

8. **Robustness** — Assume imperfect inputs and partial failure. Validate boundaries, handle errors explicitly, and fail safely rather than silently.

9. **Data over Logic** — Prefer expressing complexity in data structures, schemas, and configuration rather than in deeply nested control logic.

10. **Familiarity** — Build on established conventions and user expectations (CLI behavior, exit codes, file formats, terminology). Do not invent new conventions without strong justification.

11. **Minimal Output** — Avoid unnecessary output. Default output should be intentional, stable, and machine-parsable when appropriate. Separate human-facing verbosity from programmatic output.

12. **Diagnosable Failure** — When failures occur, emit clear, actionable error messages and meaningful exit codes. Failures should be easy to locate and reason about.

13. **Developer Time > Machine Time** — Prioritize maintainability and correctness over micro-optimizations. Optimize only when performance constraints are measured and demonstrated.

14. **Automation over Repetition** — When patterns repeat, prefer code generation, templates, or declarative definitions over manual duplication.

15. **Prototyping First** — Prototype to discover constraints and validate assumptions before polishing or optimizing. Expect early implementations to be revised or discarded.

16. **Flexibility and Openness** — Design components to be reusable and interoperable. Prefer standard interfaces, configuration-driven behavior, and loose coupling.

17. **Extensibility** — Design programs and protocols with future extension in mind. Support versioning, optional fields, and backward-compatible evolution.

### Guidance for AI Assistants

When trade-offs arise:
1. Prefer **correctness and diagnosability**
2. Then **clarity and modularity**
3. Then **performance**, only when required by evidence

Avoid introducing complexity that cannot be clearly justified or observed at runtime. Never swallow exceptions silently—always log or emit structured errors.

### Concrete Implementation Standards

* **Logging**: Use `sdrwatch.util.logging` for structured logging. Prefer `logger.exception()` in except blocks over silent `pass`.
* **Exit Codes**: Use `sdrwatch.util.exit_codes.ExitCode` constants for meaningful process exit codes.
* **Error Emission**: Use `ScanLogger.emit_error()` for structured JSONL error events alongside Python logging.
* **Observability**: Debug endpoints at `/api/debug/*` expose health, DB stats, config, and errors. The `/debug` page provides a developer dashboard.

---

## Layered runtime model (baseline-oriented scanner ↔ controller ↔ web)

* **Scanner (`sdrwatch.py`)** owns SDR/DSP loops and now operates in a **baseline-first** flow: every scan is tied to an active baseline (`--baseline-id`). Each sweep loads the baseline, runs PSD/CFAR, updates **baseline EMAs/occupancy**, persists detections into `baseline_detections`, writes lightweight `scan_updates`, and handles spur calibration into `spur_map`. It still exports built-in profiles via `--list-profiles` and can emit JSONL per detection tagged with `baseline_id`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BPFLNALCR/SDRwatch](https://github.com/BPFLNALCR/SDRwatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
