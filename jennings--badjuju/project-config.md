---
trigger: always_on
description: This file provides instructions and context for AI coding agents working on this project.
---

# Project Instructions for AI Agents

This file provides instructions and context for AI coding agents working on this project.


## Workflows

Ticket-driven workflows live as skills:

- **Working on a ticket** — `.claude/skills/work-on-ticket/SKILL.md`
- **Planning and creating tickets** — `.claude/skills/plan-tickets/SKILL.md`

Claude Code auto-loads these by description match; the skill content is
authoritative.

## Build & Test

> Don't have `redo` installed? Run `./do <target>` instead of `redo <target>`
> anywhere below. The repo ships `./do` as a self-contained shell-script
> fallback — no install required.

```bash
# Install JS dependencies
pnpm install

# Build all packages
redo all              # or just: redo  (or: ./do all)

# Run all tests
redo test

# Build the VS Code extension only
redo clients/vscode/all

# Format all code (biome + cargo fmt)
redo fmt

# CI-equivalent check (no writes: fmt-check + clippy + test + biome)
redo check
```

## Testing

**Run tests after every unit of work.** Before labeling an issue "implemented"
or committing, you MUST run `redo test` and `redo check`, and confirm all tests
pass with no warnings. Never close an issue or commit with a failing or skipped
test.

**Test runs should finish in under 5 minutes.** If `redo test` or `redo check`
has been running longer than 5 minutes, something is wrong — kill the task,
investigate the cause (hung subprocess, infinite loop, runaway test fixture),
and surface what you found before retrying. Do not just wait it out.

### Rust testing conventions

- All pure logic lives in modules (`jj.rs`, `commands.rs`, `workspace.rs`) with `#[cfg(test)]` blocks at the bottom of the same file.
- Tests that need a real `jj` repo use `tempfile::tempdir()` and call `jj git init` via `std::process::Command`. The `jj` binary is expected to be on PATH.
- Tests that call `jj` commands must use a fresh tempdir per test — never share state between tests.
- Errors must be tested, not just the happy path. For any function that returns `Result`, add at least one test that exercises the error case.
- Do not mock `jj` subprocess calls. Tests run against the real binary; that's the point.

### What to test for each new piece of work

| Work type | What to verify |
|---|---|
| New `Jj` method | Success with a real repo, failure without a repo |
| New `commands::run_*` function | File is written with expected headers, URI returned starts with `file://` |
| New `commands::on_*_save` function | State change is applied, no-op case is safe |
| New `workspace` logic | Discovery from subdirectory, returns `None` outside any repo |
| New LSP capability | `COMMANDS` list includes the new command name |
| Editing a shared client LSP-response path (e.g. `M.execute`) | Every command class that uses it: side-effect-free responses **and** state-changing responses. See "Touching shared client code" below. |

### Fixing a regression means our tests failed

A regression — something that used to work and no longer does — is by
definition a test-suite failure, not just a code failure. The whole point of
the test suite is to catch breaks before users do. If a user reports a
regression, the existing tests didn't cover the behavior they relied on, or
they covered it in a way that didn't actually exercise the broken path.

When fixing any regression:

1. **Identify the testing gap first, before writing the fix.** Ask: what
   test, had it existed, would have failed on the commit that introduced
   this bug? Why didn't we have that test? Common answers — and what they
   imply about how we write tests going forward:
   - *"The test covered the happy path but not this variant."* → We're
     under-testing variants. When adding a test, enumerate the variants
     (refocus-only vs state-changing, virtual vs physical client, repo
     present vs absent) and cover each.
   - *"The test mocked the thing that broke."* → We mocked something we
     shouldn't have. Rewrite against the real subprocess / real buffer /
     real client.
   - *"The behavior was only tested end-to-end by a human."* → Add an
     automated test at the layer where the bug lives. Manual smoke tests
     don't run in CI.
   - *"A shared code path silently changed semantics for callers that
     weren't in the test."* → Iterate over a hard-coded list of callers
     in the test, so adding a new caller is one line and a future
     short-circuit fails *every* entry. See "Touching shared client
     code" below for the canonical example (issue #72).
2. **Add the missing test, and make sure it fails on the pre-fix code.**
   Check out the broken commit, run the new test, confirm it fails. If
   it passes against broken code, it's not testing what you think it is.
3. **Then write the fix.** The test should now pass.
4. **In the commit message, name the testing gap.** Not just "fixes X"
   — say *why X wasn't caught*, so the lesson sticks for the next person
   (including future-you) reading `jj log`.

If you find yourself fixing a regression without being able to articulate
the testing gap, stop and think harder. "I just forgot to test this" is
rarely the real answer — usually there's a structural reason the test was

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jennings/badjuju](https://github.com/jennings/badjuju) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
