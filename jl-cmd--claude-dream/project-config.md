---
trigger: always_on
description: <!-- SYNC-HEADER-START -->
---

<!-- SYNC-HEADER-START -->
<!--
AUTO-GENERATED — DO NOT EDIT.
Source of truth: jl-cmd/claude-code-config/.github/copilot-instructions.md
Synced by: .github/workflows/sync-ai-rules.yml
Source commit: 9696b07e09082386d0005920e3aa1970f2c2e9ff
Synced at: 2026-06-15T00:29:31.744881+00:00
-->
<!-- SYNC-HEADER-END -->

# Code rules for Claude, Cursor BugBot, Copilot, and other agents

This file is the **canonical** review-criteria instruction set for every AI agent that audits pull requests in this repository:

- **Claude** (PR review)
- **Cursor BugBot** (PR review)
- **GitHub Copilot** (PR review)
- Any other agent that loads `AGENTS.md` or `.cursor/BUGBOT.md` for review

These rules describe the green-light state of code in this repository. Agents apply them to the **lines a PR adds or modifies**, surface deviations as findings, and recommend corrections. Output is review feedback.

Where a rule lists exemptions (test files, migrations, config files), the exemption applies. Where a rule shows a before/after pair, the "after" form is the green-light pattern.

This file is **rules-only**. Repo layout, build commands, and workflow guidance live elsewhere.

---

## Contents

- [Comments](#comments)
- [Naming](#naming)
- [Magic values & configuration](#magic-values--configuration)
- [Types](#types)
- [Structure](#structure)
- [Design](#design)
- [Tests](#tests)
- [Platform and tooling](#platform-and-tooling)
- [Repo hygiene](#repo-hygiene)
- [Scope of review](#scope-of-review)
- [Hook enforcement](#hook-enforcement)

Many bullets are implemented in `packages/claude-dev-env/hooks/blocking/code_rules_enforcer.py` (`validate_content` for Python and a small JavaScript subset). The default `PreToolUse` `Write|Edit` chain in `packages/claude-dev-env/hooks/hooks.json` registers that script alongside `tdd_enforcer.py`, `windows_rmtree_blocker.py`, the `run_all_validators` entrypoint, and others; the `Bash` chain registers `destructive_command_blocker.py`, `gh_body_arg_blocker.py`, `block_main_commit.py`, and `pr_description_enforcer.py`. **Hook enforcement** below maps each rule to its **source script** and notes Python-only coverage where it applies. Flag violations from the diff in review even when no local hook runs the same check.

---

## Code rules

### Comments

- New production code uses self-documenting identifier names. New `#`/`//` inline comments added in production code are findings; new `#`/`//` standalone comment lines and `/* ... */` block comments at line start (non-docblock) are advisory ONLY. Docstrings, `/** ... */` JSDoc docblocks, and standalone directive-marker lines (the markers listed below) are exempt. Python inline directive markers (`# noqa`, `# type:`, `# pylint:`, `# pragma:` mid-line) are also exempt; inline JS/TS directive markers (`// @ts-...`, `// eslint-...`, `// prettier-...` mid-line) remain findings.
- **IMPORTANT:** Existing comments remain exactly as written. Comments in the surrounding file are sacred.
- Docstrings on new functions, methods, classes, and modules (including module-level docstrings) are welcome.
- **Test files (`test_*.py`, `*_test.py`, `*.test.*`, `*.spec.*`, `conftest.py`) are fully exempt** — inline comments and docstrings inside test functions are welcome.
- Directive markers remain exactly as written: shebangs, `# type:`, `# noqa`, `# pylint:`, `# pragma:`, `// @ts-...`, `// eslint-...`, `// prettier-...`, and `/// ` triple-slash reference directives.

### Naming

#### Identifiers

- Identifiers use full words. Common abbreviations to expand: `ctx → context`, `cfg → configuration`, `msg → message`, `btn → button`, `idx → index`, `cnt → count`, `elem → element`, `val → value`, `tmp → temporary_value`.
- Component names describe what the component IS: `Overlay`, `Validator`, `InvoicePreview`. Generic placeholders to replace: `Screen → Overlay`, `Handler → Validator`, `Wrapper → InvoicePreview`.

#### Loop variables

- Multi-letter loop variables carry the `each_` prefix: `each_order`, `each_user`. Single-letter `i`, `j`, `k` apply to indices; `e` applies to caught exceptions.

#### Booleans, collections, and maps

- Boolean variables, parameters, and methods carry an `is_`, `has_`, `should_`, or `can_` prefix: `is_ready`, `has_payload`, `should_retry`, `can_skip`.
- Collection variables carry the `all_` prefix: `all_orders`, `all_pending_jobs`.
- Maps and dicts follow the `X_by_Y` pattern: `price_by_product`, `user_by_id`.

#### Parameters and banned names

- Direction and source parameters carry a preposition prefix: `from_path=`, `to=`, `into=`.
- Identifiers in production code describe domain meaning: `parsed_invoice`, `pending_orders`, `cached_lookup`. Generic placeholders to replace: `result`, `data`, `output`, `response`, `value`, `item`, `temp`.
- Function names use specific verbs: `parse_invoice`, `dispatch_event`, `migrate_schema`. Generic prefixes to replace (hook-enforced via `code_rules_enforcer.py::check_banned_prefixes`): `handle_`, `process_`, `manage_`, `do_`.

### Magic values & configuration

- Production function bodies use named constants for numeric, string, and boolean values. Inline literals stay acceptable for `0`, `1`, `-1`, the empty string, and `True`/`False` when the meaning is obvious.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jl-cmd/claude-dream](https://github.com/jl-cmd/claude-dream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
