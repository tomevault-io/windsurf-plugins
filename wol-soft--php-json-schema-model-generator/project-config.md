---
trigger: always_on
description: When working through a list of review notes, critically evaluate each note before acting on it:
---

# CLAUDE.md

## Responding to review notes

When working through a list of review notes, critically evaluate each note before acting on it:

- **Is the note correct?** The reviewer may be mistaken about what the code does, or may be
  operating on a false assumption. If the note is factually wrong, explain why and skip it.
- **Is the proposed fix better than the current approach?** The reviewer's suggestion is a
  starting point, not a mandate. If a different solution is clearly superior, propose it.
- **Is there an even better solution?** Think beyond the note. If the reviewer flags a smell,
  consider whether the right fix is the one they suggest or a deeper redesign.
- **Document the reasoning.** For each note, produce a summary of what action was taken and why
  — including why any note was rejected or handled differently than suggested.

After tackling all notes, provide a summary table: one row per note, action taken, and brief
reasoning.

---

## Learning from reviews

After completing a task that involved responding to code review feedback, scan the reviewer's
corrections and confirmations for patterns not already captured in memory or in this file. For
each non-obvious pattern found, write or update a `feedback` memory file in the project memory
directory and add a pointer to `MEMORY.md`.

What qualifies as worth saving:
- Any correction the reviewer had to make that I should have caught myself.
- Any expectation that surprised me or that I applied incorrectly.
- Any confirmation that a non-obvious approach was right (so it is not silently reversed later).

What does not qualify:
- One-off fixes specific to a single schema or class.
- Anything already stated verbatim in this file.
- Trivially obvious mistakes with no generalizable lesson.

Do this at the end of the session, not during — so it does not interrupt implementation work.

---

## Clarification policy

Before starting any non-trivial task — one that has more than one degree of freedom, including
architectural choices, naming decisions, scope boundaries, approach selection, or any other point
where multiple valid implementations exist — Claude must identify every such ambiguity and ask the
user to resolve it.

Rules:

- When there are multiple clarifying questions to ask, ask them **one at a time**, in order of
  dependency (earlier answers may resolve later questions). Wait for the answer before asking the
  next question. This allows the user to discuss each point in depth without being overwhelmed by
  a wall of questions.
- If new ambiguities emerge during execution that were not foreseeable upfront, pause and ask
  follow-up questions before proceeding past that decision point.
- For high-stakes decisions (architecture, scope, data model, API shape, behaviour changes) always
  block and wait for an explicit answer.
- For low-stakes decisions (minor naming, formatting, trivially reversible choices) Claude may
  proceed with a clearly stated assumption rather than blocking, but must make the assumption
  visible so the user can correct it.
- There must be no silent interpretation or interpolation of under-specified tasks. If something is
  unclear, ask. Do not guess and proceed.
- For multi-phase implementations, **never start the next phase without an explicit go-ahead from
  the user**. After completing a phase, summarise what was done and wait for confirmation before
  proceeding.

When generating a new CLAUDE.md for a repository, include this clarification policy verbatim as a
preamble before all other content.

---

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
composer update

# Run all tests
./vendor/bin/phpunit

# Run a single test file
./vendor/bin/phpunit tests/Basic/BasicSchemaGenerationTest.php

# Run a specific test method
./vendor/bin/phpunit --filter testMethodName

# Run tests with descriptive output
./vendor/bin/phpunit --testdox
```

Tests write generated PHP classes to a session-unique directory `sys_get_temp_dir()/PHPModelGeneratorTest_<id>/Models/` (defined as `MODEL_TEMP_PATH`; the base is `TEST_BASE_DIR`) and dump failed classes to `./failed-classes/` (auto-cleaned on bootstrap). The session directory is cleaned up automatically via a shutdown function when the PHP process exits.

### Running the full test suite

When running the full test suite, always save output to a file so the complete
output is available for analysis without re-running. Use `--display-warnings` to capture warning
details and `--no-coverage` to skip slow coverage collection:

```bash
php -d memory_limit=128M ./vendor/bin/phpunit --no-coverage --display-warnings 2>&1 | sed 's/\x1b\[[0-9;]*m//g' > /tmp/phpunit-output.txt; tail -5 /tmp/phpunit-output.txt
```

Then analyse with: `grep -E "FAIL|ERROR|WARN|Tests:" /tmp/phpunit-output.txt`

After analysis is complete, delete the file: `rm /tmp/phpunit-output.txt`

## Architecture

This library generates PHP model classes from JSON Schema files. The process is a 4-step pipeline:

1. **Schema Discovery** — Scan a source directory for `*.json` files (or use a custom `SchemaProviderInterface`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wol-soft/php-json-schema-model-generator](https://github.com/wol-soft/php-json-schema-model-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
