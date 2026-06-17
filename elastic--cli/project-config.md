---
trigger: always_on
description: CLI for interacting with Elasticsearch, Elastic Cloud, and Elasticsearch Serverless control plane APIs. Targets LLM-powered agents as first-class users.
---

# Elastic CLI

CLI for interacting with Elasticsearch, Elastic Cloud, and Elasticsearch Serverless control plane APIs. Targets LLM-powered agents as first-class users.

## Tech Stack

- **Runtime**: Node.js with native TypeScript (`--experimental-strip-types`)
- **CLI Framework**: Commander.js
- **Validation**: Zod v4
- **Config Management**: cosmiconfig (YAML serialization)
- **Testing**: Node.js built-in test runner (`node:test`)
- **Linting**: ESLint + TypeScript ESLint, MegaLinter (CI + pre-commit)
- **TypeScript**: Strict mode, ESNext + nodenext module resolution

Avoid adding new third-party dependencies to reduce supply-chain attack surface.

## Architecture

Commands are defined via shared config structures (see `factory.ts`). Custom logic is only permitted for behaviors that cannot be expressed in config.

## Command Authoring Requirements

All requirements below are non-negotiable and enforced at review time.

### Input

- **JSON via stdin and `--input-file`**: Structured input MUST be accepted from stdin or `--input-file <path>`. Neither takes precedence; providing both MUST error.
- **CLI flags for all input fields**: Every top-level schema field MUST have a corresponding kebab-case CLI flag. When both JSON input and flags are provided, flags take precedence.
- **Zod input schema**: Every command with structured input MUST declare a Zod schema as the single source of truth for validation, type inference, and help text. `input: true` (untyped) MUST NOT be used in new commands.
- **Validate before executing**: All input MUST be validated before any handler logic or network call. Invalid input is a hard error.
- **Reject unknown keys**: Input with undefined keys MUST produce a validation error naming the unknown field(s). Silent stripping is not acceptable.

### Output and Errors

- **`--json`**: Every command MUST emit structured JSON when `--json` is passed.
- **`--help --json`**: MUST output the full JSON Schema so agents can introspect valid inputs.
- **Errors**: All errors MUST go to stderr with a non-zero exit code. With `--json`, errors MUST serialize as `{"error": {"code": "...", "message": "..."}}`.

### Mutations and Side Effects

- **`--dry-run`**: Every command that mutates state or makes a network call MUST support `--dry-run`. In dry-run mode: validate all inputs, print the resolved request payload, exit 0 without executing.

### Credentials and Configuration

- **No credentials as CLI flags**: API keys, passwords, and tokens belong only in the config file or environment variables. Never as CLI flags.
- **Named contexts**: Connection info is managed via named contexts in the YAML config (kubectl-style). `--context <name>` MAY override for a single invocation; context fields MUST NOT be duplicated as first-class flags.

### Transport Abstraction

- **Hide routing metadata**: `found_in: path | query | body` is an implementation detail. It MUST NOT appear in help text, schema output, or error messages.
- **Validate path parameter coverage**: If a schema field has `found_in: "path"` but the URL template has no matching placeholder, the system MUST fail fast at registration time.

### Cross-Platform Compatibility

- **Paths**: Use `path.join()` / `path.resolve()`. Hard-coded `/` or `\\` separators are forbidden.
- **Config directories**: Resolve using `os.homedir()` and `process.env.APPDATA` (or OS equivalents). No Unix-only hard-coded paths.
- **Platform guards**: Signal handling, TTY detection, and ANSI escape codes MUST be guarded behind capability checks.
- **CI**: The full test suite MUST pass on Windows, Linux, and macOS before merge.

## Code Patterns & Conventions

### SPDX Header

All code files MUST start with:

```
/*
 * Copyright Elasticsearch B.V. and contributors
 * SPDX-License-Identifier: Apache-2.0
 */
```

Note the single-asterisk `/*` opener — `scripts/check-spdx` rejects the JSDoc-style `/**` form.

### Standards

- **Docstrings**: All exported symbols in reusable utilities MUST have complete doc comments.
- **Comments**: Explain WHY, not WHAT. Do not restate code in prose.
- **Naming**: camelCase for functions/variables, PascalCase for types/interfaces.
- **YAML config keys**: Always `snake_case` (e.g. `api_key`, `current_context`). Never camelCase or kebab-case.
- **Files**: Trailing newline required, no trailing whitespace.

### Dependencies

When solving a problem outside this tool's core domain (Elastic API interaction), check if an installed dependency solves it before writing custom code. For example: prefer `commander` over `process.argv` for argument parsing. Apply a manual solution only if the dependency cannot help.

### TypeScript Configuration

Strict flags: `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `strict`, `verbatimModuleSyntax`, `isolatedModules`, `noUncheckedSideEffectImports`, `moduleDetection: force`. Source maps and declaration maps enabled.

## TDD Discipline

Follow this cycle autonomously for every change:

1. Write a failing test (RED)
2. Confirm it fails for the right reason
3. Write minimum code to pass (GREEN)
4. Refactor under green (REFACTOR)

Task is complete when `npm test` exits 0 and lint passes.

## Pre-commit Linting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elastic/cli](https://github.com/elastic/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
