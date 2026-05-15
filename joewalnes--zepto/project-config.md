---
trigger: always_on
description: **This is a living document.** Update it as the project evolves — new rules, workflow changes, communication preferences, and lessons learned.
---

# Claude Code Guidelines for Zepto Editor

**This is a living document.** Update it as the project evolves — new rules, workflow changes, communication preferences, and lessons learned.

---

## The 8 Rules

Every commit must satisfy all of these. No exceptions.

### 1. Build integrity

The compiled `./zepto` must run correctly and depend only on the Perl standard library.

- `make build` must succeed
- The built binary must be self-contained — no CPAN modules, no external files
- Verify basic operation after every non-trivial change (see Testing Workflow below)
- Architecture and bundling: `DESIGN.md`, `build.pl`

### 2. UI discoverability

Every feature must be discoverable through the UI without reading help, docs, or source code.

- All features must be accessible via command palette (`⌃Space`) and/or status bar pills
- This **must** be verified interactively — run the program and check with your own eyes
- "It's just a bug fix" does **not** exempt you from interactive testing. Any change to key handling, commands, or rendering is a UI change.
- Full UI standards: `docs/UI_GUIDELINES.md`

### 3. Tests and lint pass, with no noise

- `make test` must pass completely
- `make check` (Perl syntax check) must pass
- No unexpected output on stdout/stderr during test runs
- Tests must be fast — slowness is a bug
- Full testing standards: `docs/CODE_QUALITY.md`

### 4. Security

This editor runs on users' desktops with access to their files. They trust it. Security matters.

- Read `docs/SECURITY.md` before touching file I/O, shell execution, or rendering
- Flag any new shell exec, path handling, or file operation for security review
- Never add network calls — Zepto is intentionally offline

### 5. Test before, fix, test after

For every bug or change:

1. **Reproduce first**: write a failing test or capture broken interactive behavior before touching code
2. **Fix it**
3. **Verify**: confirm the test now passes and interactive behavior is correct

Do not call work done without completing all three steps.

### 6. Bug tracking

Known bugs live in `bugs.md` with priorities P0–P3.

- **P0**: Data loss, crash, or fundamentally wrong behavior
- **P1**: Significant usability issue
- **P2**: Polish — inconsistency or minor misbehavior
- **P3**: Cosmetic / edge case

When you find a bug (even while working on something else), add it to `bugs.md` immediately. During a bug bash: work through bugs in priority order, fix and verify each one before moving to the next.

### 7. Code quality

- Follow established conventions — don't invent new patterns without a good reason
- Audit code for quality, architecture, and consistency as you work
- Full standards and ongoing audit status: `docs/CODE_QUALITY.md`

### 8. QA plan is kept current

The `qa/` directory contains the end-to-end QA test plan — one test case
per user-visible behavior. It is the executable spec that a QA engineer
(or future you) uses to validate a release.

**Every new feature, bug fix, or behavioral discovery must have a QA script. No exceptions.**

This means:
- **New feature**: add an executable test script in `qa/scripts/tier1/` AND
  a test case to the appropriate `qa/NN_*.txt` file covering happy path,
  key edge cases, and how to verify the feature is discoverable from the UI.
- **Bug fix (including regressions)**: add a test script that reproduces
  the bug AND a `QA-REG-###` entry to `qa/40_regression_bugs.txt` with a
  cross-ref to the primary feature test.
- **Behavioral discovery**: when you learn something non-obvious about
  how Zepto behaves (from code reading, user report, or interactive
  testing), add a test script and test case so it can't regress silently.
- **Always update `qa/CATALOG.md`**: add the new test ID under the right
  file's section. IDs are stable — never renumber. To retire a test,
  mark it `[RETIRED]` in place rather than deleting.
- **Run `make qa`** to verify the new test passes along with all existing tests.

Work is not done until the QA script exists and passes. "I'll add the test later" is not acceptable.

Test IDs are `QA-<TAG>-<NNN>` where `<TAG>` is the 3-6 char feature tag
listed in `qa/CATALOG.md`. Use the next unused number within that tag.

Full plan: `qa/README.md`.

---

## Cross-platform compatibility

Zepto targets macOS, Linux, and most Unix-like systems. The only runtime dependency is Perl (core modules only — no CPAN). Testing requires a few additional tools (see below).

### Runtime

- **Perl standard library only.** No CPAN modules — Zepto ships as a single self-contained file.
- **No platform-specific system calls or paths.** Use portable Perl idioms. Avoid anything that assumes a specific OS layout.

### Code and scripts

- **No hardcoded paths.** Never embed `/Users/joe/...` or any machine-specific path. Use `$PWD`, `$OLDPWD`, `$HOME`, `$(dirname "$0")`, or variables.
- **No platform-specific commands without guards.** `sed -i ''` is macOS-only; GNU `sed -i` has no empty argument. Use a platform check (`if [[ "$(uname)" == "Darwin" ]]`) or avoid the divergent syntax entirely. Same applies to `stat`, `readlink`, `mktemp` flags, clipboard commands, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joewalnes/zepto](https://github.com/joewalnes/zepto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
