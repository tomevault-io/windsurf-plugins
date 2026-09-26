---
trigger: always_on
description: This repository is the product and release home for LibreEcho: landing content,
---

# LibreEcho Product Repository — Agent Instructions

This repository is the product and release home for LibreEcho: landing content,
cross-repository public documentation, release notes, and release tooling. Read
`CONTRIBUTING.md` for change-ownership routing before non-trivial work.

## Agent operating contract

These rules govern normal task execution, subject to higher-priority system and
developer instructions and the repository/workspace safety boundaries below.

1. The user's current request defines the intended scope. A request to fix,
   implement, or update explicitly authorizes reversible local preparation in
   that scope: create or use a purpose-named branch/worktree, investigate the
   necessary in-scope files, make the required edits, create local commits, and
   run appropriate validation. Do not stop merely because the work needs more
   in-scope files than first expected; expected file count is not a boundary.

2. Work only on the named problem and the files needed to solve it. Do not fix
   adjacent bugs, refactor nearby code, clean up unrelated issues, or improve
   the architecture unless asked. Preserve unrelated dirty changes and other
   worktrees; do not reset, stash, clean, switch, delete, or absorb them merely
   to make the requested task convenient.

3. Before editing, identify the requested outcome, likely files, and validation
   needed to establish it. Ask before a material scope expansion, destructive or
   irreversible operation, or unresolved decision; first complete independent
   authorized work that does not depend on that decision.

4. Reviews, diagnoses, explanations, and plans do not authorize project-file
   edits unless the user explicitly asks for edits. Publication, push, pull
   request creation, merge, release/tagging, deployment, flashing, rebooting,
   and other hardware-changing actions remain separate explicit gates.

5. Every command must have a bounded timeout. Bounded background work,
   watchers, or servers are permitted only when needed for the authorized
   outcome; define their scope and timeout, capture their result, and stop and
   clean them up on completion or timeout. Do not retry indefinitely.

6. Run focused, meaningful checks that establish every acceptance criterion and
   all applicable repository-required checks. Do not impose a one-test limit or
   a blanket aggregate-suite requirement: broaden or repeat validation when
   changed paths, failures, required checks, or unresolved risk justify it. If
   authorized PR publication is part of the workflow, required CI must complete
   and be green on the exact head; otherwise do not poll remote CI unprompted.
   Keep host, CI, image/release, and hardware evidence distinct. Stop when every
   acceptance criterion is evidenced and required checks pass. Report unrelated
   failures and whether they block acceptance; do not fix them silently. Avoid
   tests that merely mirror reversible, low-impact implementation details.

7. Subject to higher-priority system/developer instructions and explicit
   project safety boundaries, the current request takes precedence over
   procedural defaults in skills or later non-safety guidance. If an instruction
   blocks completion, identify its exact source file and section, quote the
   exact blocking rule, explain the conflict, and complete independent
   authorized work rather than silently abandoning it.

8. Delegate independent, bounded research, review, or test analysis when tools
   support it and delegation improves speed or quality. Give each delegate
   disjoint write ownership or read-only work; never permit shared writes, and
   verify delegated findings before reporting success. Do not require recursive
   delegation when the harness does not support it.

9. Use concise plain language. Give brief progress updates during long tasks;
   report results first, with exact evidence and remaining blockers at a
   proportionate level of detail.

# Branching, Pull Requests, and Versioning

These rules govern how changes flow through this repository. They apply equally
to human contributors and AI coding agents (Claude Code, Codex, Hermes, Cursor,
Copilot, and any similar tool). Repository-specific build, test, and safety
rules elsewhere in this repository remain in force unchanged; this section
governs branch, PR, and release flow only.

## Branch model

```text
feature/<purpose>   fix/<issue-or-purpose>
       |                   |
       | PR                | PR
       v                   v
release/X.Y.0        release/X.Y.Z
 (major release)      (minor release)
       \______  _________/
               v
              main         (release branch merges back when release-ready)
```

- `main` is the integration branch. Nothing is committed directly to `main` or
  to any `release/*` branch; every change lands through a pull request.
- **Features** start from the current `main` (or from the active release branch
  when the feature is scoped to that release) on a branch named
  `feature/<short-purpose>` and merge into the next **major release branch**,
  `release/X.(Y+1).0` (for example `release/0.14.0`).
- **Fixes** follow the same flow and merge into the corresponding **minor

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aslater3/LibreEcho](https://github.com/aslater3/LibreEcho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
