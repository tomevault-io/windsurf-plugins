---
trigger: always_on
description: This document provides guidelines for AI coding agents to work efficiently in this repository.
---

# Repository Instructions for Coding Agents

This document provides guidelines for AI coding agents to work efficiently in this repository.

Refer to [`CONTRIBUTING.md`] for additional details.

## Communication Guidelines

The maintainer is Japanese and may communicate with you in Japanese, but source code and documentation should be written in **English**.
If you are addressed in Japanese, please conduct the conversation in Japanese.

## Project Overview

`eslint-plugin-markdown-preferences` is an ESLint plugin designed to enforce consistent style and formatting in Markdown files. Key features include:

- Automatic formatting and style consistency for Markdown
- Conversion of specific words into inline code or links
- Flexible configuration options

## Proposing New Rules

- Provide a clear description of the rule, its purpose, and any specific requirements.
- Note that this project focuses on rules related to Markdown preferences. Ensure your proposed rule aligns with this scope.
  - Rules that already exist in [`@eslint/markdown`] are not accepted.
  - Rules that might be accepted by [`@eslint/markdown`] should be proposed there first. If not accepted, they may be added to this package.
- Please carefully check the existing rules in this plugin and do not propose rules that already exist.

## Development Workflow

### General Quality Standard

For every task, make a deliberate best effort to prevent mistakes.

Before finalizing work, re-read the user's request and all applicable instructions, inspect the complete result in context, challenge your assumptions, and evaluate correctness, simplicity, consistency, edge cases, unintended effects, maintainability, and validation adequacy.

Actively look for problems beyond explicit requirements, examples, checklists, tests, and automated checks. Do not treat any checklist or passing test suite as an exhaustive guarantee. Investigate remaining uncertainty and resolve every known concern before claiming completion.

### Adding a New Rule

1. Use `npm run new -- <rule-name>` to generate a template for the new rule.
2. Edit the following files to implement the rule:
   - Implementation: `src/rules/<rule-name>.ts`
   - Tests: `tests/src/rules/<rule-name>.ts`
   - Test fixtures: `tests/fixtures/rules/<rule-name>/`
   - Documentation: `docs/rules/<rule-name>.md`

When editing files, please note the following:

- Do not use the `any` type when implementing rules.
- Please use the provided utilities as much as possible when implementing rules.
- Respect the format of the initially generated documentation.
  - In particular, ensure that sample code and the order of sections are consistent with other documents.

### Rule Testing

#### Fixture-first Rule Test Policy

- Rule behavior tests must use fixtures under `tests/fixtures/rules/<rule-name>/` and load them through the repository's fixture-based test utilities.
- Do not add inline `valid` or `invalid` rule test cases when the same case can be represented by a fixture.
- Use a non-fixture testing approach only when the fixture infrastructure cannot express the test. In that exceptional case, add a comment explaining why a fixture cannot be used and keep the non-fixture test as small as possible.
- Direct unit tests for extracted, general-purpose utilities are not rule behavior tests and may be written without rule fixtures.

#### Creating Test Fixtures

Test fixtures for rules should be organized as follows:

```plaintext
tests/fixtures/rules/<rule-name>/
├── invalid/             # Invalid cases
│   ├── xxx-input.md
│   ├── yyy-input.md
│   └── ...
└── valid/               # Valid cases
    ├── xxx-input.md
    ├── yyy-input.md
    └── ...
```

- Name input files descriptively, e.g., `multiple-heading-input.md`, `setext-heading-input.md`, etc.
- Place configuration files as `_config.json` in the same directory. You may also use `xxx-config.json` for input-specific (`xxx-input.md`) configs.
- **Configuration File Format:** All rule configuration files must follow the ESLint format with an `options` array:
  ```json
  {
    "options": [{ "mode": "fixed-line-length", "length": 30 }]
  }
  ```
  For complex configurations:
  ```json
  {
    "options": [
      {
        "listItems": {
          "first": 4,
          "rest": 2
        }
      }
    ]
  }
  ```
  **Important:** Do not use the direct format like `{ "mode": "fixed-line-length" }` as it will result in "Unexpected key" configuration errors during testing.
- You may create subdirectories such as `multiline/` or `spaces/` for different test cases.

  **Example:**

  ```plaintext
  tests/fixtures/rules/<rule-name>/
  ├── invalid/
  │   ├── multiline/
  │   │   ├── _config.json
  │   │   ├── xxx-input.md
  │   │   └── ...
  │   ├── yyy-input.md
  │   └── ...
  └── valid/
      ├── spaces/
      │   ├── _config.json
      │   ├── xxx-input.md
      │   └── ...
      ├── yyy-input.md
      └── ...
  ```

- If possible, it is recommended to add a level 1 heading at the beginning of each test case to describe the fixture.

#### Running Tests

Run tests with:

```sh
npm test
```

To update snapshot files, run:

```sh
npm run test:update
```

Ensure that snapshot files reflect the correct results.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ota-meshi/eslint-plugin-markdown-preferences](https://github.com/ota-meshi/eslint-plugin-markdown-preferences) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
