---
trigger: always_on
description: This file is the primary repository instruction file for AI coding agents. It applies to the
---

# EvalEx agent guide

## Purpose and scope

This file is the primary repository instruction file for AI coding agents. It applies to the
entire repository unless a more specific `AGENTS.md` exists below the directory being changed.
Tool-specific instruction files, such as `CLAUDE.md`, should reference this file instead of
duplicating it.

Use this guide together with the issue, `README.md`, `SECURITY.md`, the documentation under
`docs/`, the Maven configuration, and relevant tests. Do not make this file the only source of a
rule that human contributors also need to follow. When a durable project practice changes, update
the relevant human-facing documentation and this guide together.

EvalEx is a compact Java library for parsing and evaluating expressions. It supports numbers,
booleans, strings, date and time values, durations, arrays, structures, `NULL`, variables, custom
functions and operators, implicit multiplication, and lazy function parameters. Numerical
calculations use `BigDecimal`. The minimum supported Java version is 11, and the library has no
runtime dependencies.

## Human responsibility and AI transparency

- AI assistance does not reduce the contributor's responsibility. A human contributor must
  understand, review, and take responsibility for every submitted change.
- Do not publish, merge, approve, release, or push changes unless a maintainer explicitly asks for
  that exact action. Prepare local changes for human review by default.
- Do not sign commits, add a `Signed-off-by` or `Co-authored-by` trailer for an AI tool, or claim
  that generated output is original or free of third-party material.
- Disclose material AI assistance in the pull request description. Name the tool and model when
  known, and summarize what it produced or reviewed. The human contributor remains the author and
  decision maker.
- Never submit output that the contributor cannot explain. Treat AI-generated code, tests,
  documentation, citations, and analysis as untrusted until checked against the repository and
  authoritative sources.
- Do not send source code, issue details, logs, credentials, personal data, embargoed
  vulnerabilities, or other non-public repository information to an external service unless the
  contributor has confirmed that the service and its terms are approved for that data.

## Repository layout

- `src/main/java/com/ezylang/evalex/`: public entry points and implementation.
- `src/test/java/com/ezylang/evalex/`: JUnit 5 tests, generally mirroring production packages.
- `docs/`: user documentation published through GitHub Pages.
- `pom.xml`: Java level, dependencies, build plugins, formatting, coverage, and publishing setup.
- `spotless/header.txt`: required license header template for Java files.
- `.github/workflows/build.yml`: required CI build and consumer POM checks.
- `.github/workflows/publish.yml`: maintainer-only release workflow.
- `SECURITY.md`: vulnerability reporting process and security scope.

Do not edit generated content under `target/` or `.flattened-pom.xml`. Regenerate it through Maven
when it is needed for verification.

## Working method

1. Read the complete issue or task and inspect `git status` before editing.
2. Read the affected production code, its tests, and relevant documentation. Search for existing
   abstractions and conventions before adding new ones.
3. Keep the change limited to the requested behavior. Avoid speculative refactoring, unrelated
   cleanup, broad reformatting, or generated churn.
4. Preserve all pre-existing changes. Do not overwrite, revert, stage, or commit work that is not
   part of the task.
5. For a bug fix, first add or identify a focused regression test and observe the failure when
   practical. Then implement the smallest complete fix.
6. Run focused checks while developing and the complete required checks before handing off.
7. Review the final diff for correctness, unintended API changes, missing tests, documentation,
   security, licensing, and unrelated changes.
8. Report changed files, verification performed, and any check that could not be run. Never imply
   that a check passed when it was not run successfully.

Ask before making a destructive change, installing system software, changing repository or
account settings, or performing a remote write. Do not create commits or branches unless the task
requests them. If the task conflicts with this guide or lacks a decision that would materially
change the public contract, stop and ask the maintainer.

## Build and verification

### Prerequisites

- Use JDK 11 compatibility. A newer JDK may run Maven, but production code must compile with
  `--release 11` and must not use APIs introduced after Java 11.
- Use Maven from the repository root. This repository currently has no Maven wrapper.

### Commands

- Focused test class: `mvn --batch-mode -Dtest=ClassName test`
- Focused test method: `mvn --batch-mode -Dtest=ClassName#methodName test`
- Apply Java formatting and license headers: `mvn --batch-mode spotless:apply`
- Check formatting: `mvn --batch-mode spotless:check`
- Complete clean verification: `mvn --batch-mode clean verify`

Run focused tests first for fast feedback. Before review, run `mvn --batch-mode clean verify`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ezylang/EvalEx](https://github.com/ezylang/EvalEx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
