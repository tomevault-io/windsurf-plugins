---
trigger: always_on
description: This file (`AGENTS.md`) is the canonical agent-instructions document.
---

# Claude and Other Agents

This file (`AGENTS.md`) is the canonical agent-instructions document.
`CLAUDE.md` is a symlink to it for cross-tool compatibility — edit `AGENTS.md`
and refer to it by that name.

# Guidelines

- If an LSP / symbol-navigation tool is available, prefer it over Grep/Read for
  code navigation — use it to find definitions, references, and workspace symbols.
  Fall back to Grep/Read when no such tool is exposed.

- CRITICAL: NEVER suppress warnings, lint errors, or test failures with annotations
  (`#[allow(...)]`, `noqa`, `@SuppressWarnings`, etc.) unless the suppression itself
  is the correct semantic choice (e.g., `#[allow(clippy::too_many_arguments)]` on a
  function that genuinely needs many parameters). If a lint fires, fix the underlying
  code. If a test fails, fix the bug. If a warning appears, resolve the root cause.
  Quick workarounds that hide problems are strictly prohibited.

- CRITICAL: Always prefer proper solutions over quick fixes.
  When facing a problem, invest the effort to understand the root cause and fix it
  correctly rather than applying a workaround. Specifically:
  - Do NOT skip platforms or configurations to avoid fixing a build failure.
  - Do NOT add conditional compilation or feature gates to hide broken code.
  - Do NOT remove or weaken tests to make CI pass.
  - Do NOT add TODO/FIXME/HACK comments as a substitute for doing the work now.
  - If a proper fix requires changing multiple files or modules, do it.
  - If a proper fix requires understanding unfamiliar code, read it first.
  - If you are unsure whether a fix is proper, ask before proceeding.

- IMPORTANT: Don't worry about breaking compatibility or being backwards compatible.
    - neither for the API nor for the Wire Format
    - FTM this software has not been make public yet and there is no system using it. 
    - Keep the code simple.

- IMPORTANT: when planing and before you do any work:
  - ALWAYS mention how you would verify and validate that work is correct
  - include TDD tests in your plan
  - take a behaviour driven approach
  - you are very much ENCOURAGED to ask questions to get the design correct
  - ALWAYS seek clarifications to sort out ambiguities
  - ALWAYS provide a summary of the Design and implementation Plan

- IMPORTANT: when you build code and new features:
  - ALWAYS document those features in docs/
  - Remember to add examples (see below)

- IMPORTANT:
  - when you commit your work, make sure it passes all checks, tests and lints -- by running `make all`

- IMPORTANT: Derived artefacts have one source of truth.
  If a file is produced from another file or tool invocation — codegen output,
  schema-generated bindings, vendored headers, generated docs, lockstep API
  maps, compiled assets — do not maintain an unmanaged copy elsewhere.
  Consumers should read from the canonical generated location, or the
  duplicate must be guarded by a CI step that regenerates from source and
  fails on any drift. The same rule applies to values that are conceptually
  one piece of data but appear in many places (release version, ABI version,
  wire-format version, error code lists, dtype tables): pick one source of
  truth and either generate the others from it or add a consistency check.

- IMPORTANT: Build-system wrappers must not replace the inner tool's
  freshness logic.
  When CMake / Make / Bazel / shell scripts / CI jobs invoke another build
  tool (Cargo, npm, maturin, wasm-pack, Gradle, Docker, another Makefile),
  the wrapper must either invoke the inner tool unconditionally or declare a
  complete dependency graph for it. Do not key freshness solely on "output
  file exists" — the inner tool already knows what is stale, and an mtime
  check from the outer tool can serve a days-old artefact to today's
  consumers with no warning. Prefer always-running wrapper targets
  (e.g. `add_custom_target` over `add_custom_command(OUTPUT ...)` when the
  command is a nested incremental build tool); use output-file rules only
  for truly declarative single-input → single-output generators with all
  inputs listed in `DEPENDS`.

- IMPORTANT: NEVER use process-ephemeral references in code, comments, docstrings,
  commit messages, or planning documents. The code outlives the workflow that
  produced it; references to that workflow age into noise.

  Banned vocabulary (each becomes meaningless once the PR merges):
  - Workflow ordinals: "Phase 2", "Pass 5", "Round 1", "Step 3"
  - Issue-tracker references: "PR-1", "sub-task 4", "issue #94"
  - Review-feedback severity buckets: "Critical #1", "High #2"
  - History phrases inside code: "before this PR", "after review feedback",
    "fixed in commit abc1234"

  Replace each with a name that describes what the thing **is** or **does**:
  - "Round 1 paired fetch" → "the paired postamble fetch"
  - "Phase 2 state refactor" → "extracting the bidirectional state machine"
  - "Pass 5 polish" → "documentation cleanup"
  - "Critical #1 invariant" → "`fwd_terminated` cascades to `disable_backward`"

  Git history records chronology; comment and commit text must record
  substance. A future maintainer reading "Round 2" or "Phase 4" gains
  nothing — those ordinals reference the author's mental sequence at write

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecmwf/tensogram](https://github.com/ecmwf/tensogram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
