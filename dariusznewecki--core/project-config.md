---
trigger: always_on
description: manages the asyncio loop, injects services, and formats `ActionResult` output. Mutating
---

# CLAUDE.md — CORE

Loaded automatically at the start of every Claude Code session. Read fully before touching code.

---

## What CORE is, and what you are

CORE is a constitutionally-governed software factory: a runtime that supervises AI code
generation under deterministic rules. `.intent/` is law (read by the runtime); `.specs/` is
architectural reasoning (read by humans); `src/` is the implementation.

You (Claude Code) are the execution arm, not the governor. The human owns this repo, writes
intent, and reviews outputs — they do not write code manually. Your job is to produce correct,
complete files that conform to `.intent/` and the decisions in `.specs/decisions/`. AI output
is not trusted by default — it is verified. Produce work that earns the verification.

This file is a **development contract**, not a runtime governance posture. CORE is
bootstrapped on itself; the restrictions here govern how Claude Code works on this repo and
are intentionally permissive under governor confirmation. The strong version — what CORE
enforces on governed projects at runtime — lives in `.intent/`. Do not import that live
strictness into this dev contract, and do not export this dev permissiveness into runtime rules.

---

## Source layout

```
src/
  api/        — FastAPI routes and dependency providers only. No business logic.
  body/       — Analyzers, atomic actions, services, infrastructure workers. Execution layer.
  cli/        — Typer CLI commands. Rich rendering allowed here only.
  mind/       — Constitutional logic engines (ast_gate, glob_gate, llm_gate, runtime_gate, …).
                Reads .intent/ at runtime. No I/O, no execution, no Body/Will invocation.
  shared/     — Cross-cutting substrate. Must not import src/mind/, src/body/, or src/will/.
  will/       — Autonomous developer, cognitive orchestration, agents.
.intent/      — Governance law as data (YAML/JSON). Read at runtime via IntentRepository;
                never imported as Python.
.specs/       — Charter, northstar, papers/ (47+), requirements/ (URS), decisions/ (ADRs),
                planning/ (roadmaps).
```

Mind/Body/Will names *code* layers; `.intent/` and `.specs/` are *data* surfaces. Do not
conflate "the Mind" (`.intent/`) with "the Mind layer in code" (`src/mind/`).

**ADRs are live.** Authored by the human before implementation, then implemented as a
change-set. If an ADR is referenced in a prompt, read it before editing. Before editing a
layer or subsystem you have not seen before, check `.specs/papers/` and `.specs/decisions/`.

---

## How to work in this repo

**Reconnaissance before editing.** For any file/module not seen this session: read it, report
what you found, identify every affected call site — then edit. For non-trivial changes
(multi-file, schema changes, new fields on shared models, >3 files) pause after
reconnaissance for governor review.

**Tests are part of the change.** Signature/behavior change in `src/` → update the
corresponding test in the same change. New public function/class → at least basic tests. You
author the tests and run them scoped to the file(s) you touched (e.g. `pytest
tests/path/to/test_file.py -q`) — full-suite and shared-state runs still route through the
governor (see Verification after editing). Do not rely on the autonomous test-gen loop to
compensate: it is deliberately scope-limited to a pilot set (`include_files` in
`.intent/enforcement/config/test_coverage.yaml` — 44 files as of 2026-07-16, mostly
`will/workers/` + `api/v1/` routes), so it does not cover most of `src/`. Its write-time
gates have since hardened — import-resolution + shape checks (#574, #589), a
`test.sandbox_validate` execution gate that rejects generated tests which fail to run, and
pytest-in-the-loop acceptance inside the generation loop (ADR-135 D6 / ADR-140, #791) —
but until that scope opens, tests for your change are on you.
Minimum-scope does **not** exempt test updates.

**Complete files, not diffs.** Output the complete file content in a fenced block labelled
with its path. The governor reviews whole files.

**Verification after editing.** Run `ruff check` on every file touched; run import/
instantiation smoke tests where possible. Scoped `pytest` runs against the specific test
file(s) you touched or added for this change are part of normal delivery — run them and
report the result honestly, including failures. This does not extend to full-suite runs,
`-k`/directory-wide sweeps, or any test file known to hit shared live state (the `core_test`
database, a live daemon/API integration) — those remain governor-initiated; ask first. Commit
your own authored work (stage by name per ADR-101 D1, `Co-Authored-By` trailer) once verification
is reported — the governor confirmed this 2026-06-14. **Push remains governor-only**: never push
without the governor's explicit go-ahead in the current turn. Restarting `core-daemon` +
`core-api` is in-scope when a fix needs operationalization (the daemon caches imported modules);
avoid restarting mid-flight against an active CCC scan or long-running remediation.

**When in doubt, ask.** Do not invent requirements or infer CLI commands from plausibility.
If a brief says "atomic action" and no CLI surface exists to invoke one, ask rather than

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DariuszNewecki/CORE](https://github.com/DariuszNewecki/CORE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
