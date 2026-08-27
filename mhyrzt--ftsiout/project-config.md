---
trigger: always_on
description: > Project-neutral template for coding and research agents.
---

# AGENTS.md

> Project-neutral template for coding and research agents.
>
> Replace values written as `<PLACEHOLDER>` with repository-specific information.
> Delete sections that are not applicable to the project rather than leaving misleading rules behind.

## Purpose

This file defines the operating contract for AI/coding agents working in this repository.

Agents must treat repository contents as authoritative project artifacts, not as disposable text.
Changes to code, configuration, documentation, tests, schemas, generated artifacts, research claims,
data, filenames, APIs, or build structure may alter project behavior or meaning.

Before making substantial changes, inspect the actual repository and determine:

- the project structure;
- the build/test/lint workflow;
- source-of-truth files;
- generated versus hand-maintained artifacts;
- repository-local conventions;
- relevant nested `AGENTS.md` or equivalent policy files;
- whether the requested task is an audit, proposal, implementation, rewrite, migration, or release task.

Do not assume this template describes the repository more accurately than the repository itself.

---

# Core principles

1. **Do not invent facts or behavior.**
   - Never fabricate APIs, files, commands, dependencies, citations, experiment results, configuration
     values, implementation behavior, or project history.
   - If something cannot be established from repository evidence or an explicit user instruction,
     mark it unresolved or state the assumption clearly.

2. **Do not silently reconcile conflicts.**
   - If code, documentation, tests, configuration, generated output, or other artifacts disagree,
     report the conflict.
   - Resolve it only from an appropriate authoritative source or an explicit user decision.

3. **Preserve scope.**
   - Do not generalize a result, guarantee, API contract, benchmark, or implementation property beyond
     the conditions under which it is established.
   - Distinguish current behavior from intended behavior.

4. **Separate evidence from interpretation.**
   Use explicit categories where useful:
   - observed behavior;
   - verified implementation fact;
   - documented contract;
   - inference;
   - hypothesis;
   - proposal/future work;
   - unresolved issue.

5. **Preserve reproducibility and traceability.**
   Where relevant, retain:
   - commit hashes;
   - versions;
   - configuration names;
   - environment/toolchain versions;
   - seeds;
   - dataset or experiment IDs;
   - source paths;
   - generated artifacts;
   - migration history;
   - build/test commands.

6. **Prefer reversible edits.**
   - Keep unrelated refactors separate.
   - Avoid combining structural migration, behavior changes, formatting changes, dependency upgrades,
     and documentation rewrites in one opaque change.
   - Prefer small, auditable patches over broad rewrites.

7. **Respect repository-local conventions.**
   - Existing project conventions override this generic template unless they conflict with an explicit
     user instruction.
   - Do not introduce a new framework, tool, directory convention, formatter, or architecture merely
     because it is personally preferable.

---

# Instruction precedence

Use the following priority unless the repository defines a stricter policy:

1. explicit user instruction for the current task;
2. repository-local policy (`AGENTS.md`, `CONTRIBUTING.md`, architecture decisions, style guides);
3. authoritative project configuration and executable behavior;
4. established project conventions;
5. this generic template.

If two higher-priority sources conflict, stop guessing and report the conflict.

Nested policy files apply to their subtree and may refine or override this file.

---

# Repository discovery

Before substantial edits, inspect the repository rather than assuming a layout.

Record or identify, when applicable:

```text
Project root:
Primary language(s):
Package/build system:
Application entry points:
Test directories:
Documentation directories:
Configuration directories:
Generated directories:
Data/artifact directories:
CI configuration:
Release configuration:
Repository-local agent instructions:
```

Example only:

```text
src/
tests/
docs/
scripts/
config/
assets/
.github/
AGENTS.md
README.md
```

Do not create missing directories just to match this example.

---

# Source-of-truth discipline

For any factual, behavioral, numerical, or structural claim, identify the most authoritative available
source.

A typical ordering is:

```text
1. executable code / schema / canonical data
2. automated tests or validated generated artifacts
3. authoritative configuration
4. generated reports derived from canonical sources
5. maintained technical documentation
6. README / narrative documentation
7. comments, examples, slides, or historical notes
```

This ordering is only a default. A repository may explicitly define another authority hierarchy.

When artifacts disagree:

- do not silently normalize them;
- identify the mismatch;
- determine the likely source of truth;
- update dependent artifacts consistently when the task requires it.

Avoid circular authority such as treating a summary as proof of the source data it summarizes.

---

# Editing boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mhyrzt/ftsiout](https://github.com/mhyrzt/ftsiout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
