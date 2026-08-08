---
trigger: always_on
description: @/Users/raezil/.codex/RTK.md
---

@/Users/raezil/.codex/RTK.md

# AGENTS.md

## Purpose

This repository uses agent-assisted development with a disciplined Go workflow.

`go-harness` is an agentic Go coding harness built on
`github.com/Protocol-Lattice/go-agent`. It provides a CLI/REPL that loads
markdown skills, persists session memory, discovers UTCP tools, and runs coding
tasks through filesystem, shell, and git provider binaries.

The goal is to make small, correct, maintainable changes without inventing
behavior, rewriting unrelated code, or weakening tests.

## Agent Stack

Use this stack together, in this priority order:

1. **Superpowers** - stay disciplined, verify assumptions, and avoid
   overengineering.
2. **RTK** - inspect, navigate, and execute repository changes safely.
3. **grill-me** - stress-test implementation plans before coding.
4. **Ponytail** - prevent over-engineering; prefer deleting, reusing,
   simplifying, and avoiding speculative abstractions before adding code.
5. **cc-golang-skills** - write idiomatic, tested, production-quality Go.
6. **Caveman** - compress communication without losing facts, commands, paths,
   errors, or test results.

`grill-me` is the plan-review layer. It must challenge weak assumptions, missing
edge cases, risky scope, vague requirements, and untested behavior before
non-trivial implementation starts.

Ponytail is the anti-overengineering layer. It must challenge whether new
code, new abstractions, new dependencies, new files, or broad rewrites are
actually necessary before implementation.

Caveman is the communication layer, not a shortcut. It must never override
Superpowers, RTK, grill-me, Ponytail, cc-golang-skills, tests, safety, or
correctness.

## Core Rules

1. Read this `AGENTS.md` before making changes.
2. Inspect the current repository before proposing implementation.
3. Do not invent files, APIs, packages, providers, commands, or behavior.
4. Prefer simple, boring, maintainable Go.
5. Make small, reviewable changes.
6. Keep public APIs stable unless the task explicitly requires breaking changes.
7. Add or update tests for every behavior change.
8. Run formatting and relevant tests before finishing.
9. Be honest about what changed, what was tested, and what was not tested.
10. Use `grill-me` for non-trivial plans before implementation.
11. Use Ponytail before coding to prefer the smallest correct change.
12. Use Caveman only to reduce output noise; never to reduce engineering rigor.

## Required Workflow

For every non-trivial task, follow this order:

1. **Read project instructions**

   - Read `AGENTS.md`.
   - Check relevant README sections, docs, examples, tests, and package layout.

2. **Inspect with RTK**

   - Use RTK for repository inspection, navigation, and command execution.
   - Identify relevant packages, commands, tests, examples, and existing
     conventions.
   - Do not assume architecture from memory.

3. **Plan**

   - Produce a short implementation plan before coding.
   - Include the problem, goal, constraints, risks, and acceptance criteria when
     useful.
   - Prefer minimal changes.
   - Avoid broad rewrites unless explicitly requested.

4. **Review with grill-me**

   - Stress-test the plan before coding.
   - Challenge unclear requirements, hidden assumptions, missing edge cases,
     risky scope, and weak tests.
   - Revise the plan if `grill-me` finds a real gap.
   - Do not use `grill-me` to expand scope beyond the user request.

5. **Simplify with Ponytail**

   - Ask whether the change needs new code at all.
   - Prefer using existing code, standard library, native platform features,
     smaller diffs, and deletion over new abstractions.
   - Reject speculative future-proofing, unnecessary dependencies, and broad
     rewrites unless the user explicitly asks for them.
   - Use `@ponytail-review` for diff review when available.
   - Use `@ponytail-audit` for whole-repository over-engineering audits when
     available.

6. **Implement**

   - Use idiomatic Go.
   - Keep interfaces small.
   - Use explicit error handling.
   - Use `context.Context` where cancellation, deadlines, I/O, or request scope
     matter.
   - Avoid unnecessary abstraction.
   - Avoid global mutable state unless clearly justified.
   - Preserve existing behavior unless the task requires changing it.

7. **Test**

   - Add or update tests for behavior changes.
   - Prefer table-driven tests where useful.
   - Cover success paths and failure paths.
   - Do not fake test results.

8. **Verify**

   Run relevant commands through RTK:

   ```sh
   rtk gofmt -w .
   rtk go test ./...
   rtk go vet ./...
   ```

   If the change touches build, providers, CLI behavior, or module state, also
   consider:

   ```sh
   rtk make build
   rtk make test
   rtk make tidy
   ```

9. **Report**

   Summarize what changed, why it changed, files changed, tests run, and known
   limitations or follow-ups.

## RTK Usage

Use RTK as the execution and repository-navigation layer.

Always prefix shell commands with `rtk`, including read-only commands:

```sh
rtk git status
rtk rg "pattern"
rtk sed -n '1,120p' README.md
rtk go test ./...
```

Use RTK to:

- Inspect the work tree.
- Read relevant files.
- Understand current architecture.
- Locate tests and examples.
- Apply focused changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Protocol-Lattice/go-harness](https://github.com/Protocol-Lattice/go-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
