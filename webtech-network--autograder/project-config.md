---
trigger: always_on
description: This document is the authoritative architectural reference for anyone working on this codebase — human or AI agent. Read it before making any change. Every design decision described here is intentional and must be respected.
---

# Autograder — Agent & Contributor Reference

This document is the authoritative architectural reference for anyone working on this codebase — human or AI agent. Read it before making any change. Every design decision described here is intentional and must be respected.

---

## What this project is

The autograder is a **general-purpose code grading engine**. It accepts a submission (a set of files) and an assignment configuration (a grading criteria definition), runs a configurable pipeline of evaluation steps, and returns a score from 0 to 100 together with a structured result tree and optional feedback.

The core engine is deliberately domain-agnostic. It has no knowledge of GitHub, git, gamification, learning management systems, or any specific use case. Those concerns belong in the layers that sit on top of the core.

---

## Repository layout

```
autograder/          Core grading engine — the only thing that must stay general
sandbox_manager/     Infrastructure: Docker-based sandbox container management
github_action/       Adapter: runs the grader inside a GitHub Actions workflow
web/                 Adapter: FastAPI server exposing the grader as an HTTP API
tests/               Unit, integration, e2e, and web tests
```

The boundary between `autograder/` and everything else is the most important architectural line in the codebase. Code inside `autograder/` must not import from `web/`, `github_action/`, or any other adapter. The adapters import from `autograder/`, not the other way around.

---

## The two inputs: Submission and Assignment Configuration

Every grading execution starts with two independent inputs.

### Submission

A `Submission` (`autograder/models/dataclass/submission.py`) represents what is being evaluated. It carries:

- `submission_files: Dict[str, SubmissionFile]` — the files to grade, keyed by filename. Each `SubmissionFile` holds a `filename` and the full text `content`.
- `user_id` / `username` — identifiers for the submitter, used for logging and export. The autograder does not interpret or validate these values.
- `assignment_id` — an opaque identifier that links this submission to a grading configuration. The autograder treats this as a correlation ID only.
- `language: Optional[Language]` — the programming language of the submission. Required by any step that does language-specific analysis (AST parsing, sandbox execution).
- `locale: str` — controls the language of generated feedback messages.

The autograder does not fetch files, call APIs, or talk to version control. The caller constructs the `Submission` object with whatever files are relevant and passes it to the pipeline. Responsibility for understanding what files to include, and from where, belongs entirely to the caller.

### Assignment Configuration (Grading Config)

The assignment configuration is a set of JSON/dict structures passed to `build_pipeline()`:

- `template_name` — selects which `Template` (test library) to use.
- `grading_criteria` — a `CriteriaConfig` dict that defines the full scoring rubric: categories, subjects, tests, and weights. This is the central configuration artifact.
- `feedback_config` — preferences for the feedback report generator.
- `setup_config` — optional pre-flight instructions (required files, setup commands, assets to inject into the sandbox).

The criteria config is what separates one assignment from another. Two assignments using the same template can have completely different rubrics, weights, and test selections. The template provides the available test functions; the criteria config decides which ones to use and how to weight them.

---

## The Criteria Tree: the pluggable scoring engine

The criteria tree is the most important concept in the codebase. Understanding it fully is required to work anywhere in `autograder/`.

### What it is

A `CriteriaTree` (`autograder/models/criteria_tree.py`) is the compiled, in-memory representation of a grading rubric. It is built from the `grading_criteria` dict during `BuildTreeStep` and is immutable for the duration of a pipeline execution.

The tree has a fixed three-level top structure:

```
CriteriaTree
  ├── base: CategoryNode        (required — the main score, 0-100)
  ├── bonus: CategoryNode       (optional — adds points)
  └── penalty: CategoryNode     (optional — subtracts points)
```

Each `CategoryNode` can contain either:
- A flat list of `TestNode`s (leaves), or
- A list of `SubjectNode`s, each of which can recursively contain more subjects or tests.

This recursive subject structure allows arbitrary grouping and weighting of tests. A `SubjectNode` is just a weighted container — it has a `name`, a `weight`, and children that are either more subjects or test leaves.

### TestNode: the leaf

A `TestNode` is the leaf of the criteria tree. It holds:

- `test_function: TestFunction` — a reference to the actual callable that will evaluate the submission.
- `parameters: Dict[str, Any]` — the arguments that will be passed to the test function when it runs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webtech-network/autograder](https://github.com/webtech-network/autograder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
