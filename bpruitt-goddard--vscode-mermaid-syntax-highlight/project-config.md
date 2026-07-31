---
trigger: always_on
description: This repository provides Markdown syntax support for the Mermaid charting language in VS Code. It uses TextMate grammars defined in YAML and compiled to JSON.
---

# Agent Guidelines for vscode-mermaid-syntax-highlight

This repository provides Markdown syntax support for the Mermaid charting language in VS Code. It uses TextMate grammars defined in YAML and compiled to JSON.

## 1. Build, Lint, and Test

### Dependencies

- **Install:** `npm install`

### Grammar Compilation

**Crucial:** The source of truth for grammars is in `syntaxes/`. The VS Code extension uses JSON files in `out/`, which are generated from the YAML files.

- **Compile YAML to JSON:** `npm run convertYaml`
- **Watch/Auto-compile:** Not explicitly provided, so run `npm run convertYaml` manually after any change to `syntaxes/**/*.yaml`.

### Testing

Tests verify that the generated grammar correctly scopes the Mermaid code.

- **Run All Tests:** `npm test`
- **Run Single Test File:**
  ```bash
  npx vscode-tmgrammar-test "tests/diagrams/graph.test.mermaid"
  ```
- **Recommended Workflow:**
  Always compile before testing to ensure your changes are active.
  ```bash
  npm run convertYaml && npm test
  ```

### Packaging

- **Create VSIX:** `npx @vscode/vsce package`

### Formatting

- **Format Code:** `npm run format:write` - must run before any commit to ensure consistent formatting.

## 2. Code Style & Conventions

### Grammar Definitions (`syntaxes/`)

- **Format:** YAML files in `syntaxes/diagrams/`.
- **Required Tests:** Every diagram grammar file must have one or more associated test files in `tests/diagrams/`. Tests are required to validate the grammar scoping and prevent regressions.
- **Custom YAML Tag `!regex`:**
  - Use `!regex |-` for all non-trivial regular expressions.
  - This allows multiline regexes with comments, which are stripped during compilation.
  - **Structure pattern:** Split the regex into multiple lines, with each line corresponding to a capture group. This makes it easier to match the regex with the associated capture group definition.

  **Example:**

  ```yaml
  match: !regex |-
    ^\s*(subgraph)\s+ # Matches the subgraph keyword
    (\w+)             # Matches the subgraph ID
  captures:
    '1':
      name: keyword.control.mermaid
    '2':
      name: variable
  ```

  **Becomes (in JSON):** `^\\s*(subgraph)\\s+(\\w+)`

- **Scopes:** Use standard TextMate scope names where possible.
  - `keyword.control.mermaid`: Keywords like `graph`, `subgraph`, `end`.
  - `entity.name.function.mermaid`: Names of diagrams or major sections.
  - `variable`: Node IDs, variables.
  - `string`: Text content, labels.
  - `comment`: Comments (`%%`).

### Test Files (`tests/`)

- **Format:** `.test.mermaid` (or `.md`, `.mdx`) files.
- **Whitespace Awareness:** These files are whitespace-aware. You must maintain consistent tabs/spacing throughout the file to avoid test errors.
- **Scope Assertion Rules:**
  - **Every non-commented line must have corresponding scope assertions below it.**
  - **Every non-whitespace character on the test line must have a corresponding scope validation line below it.** Assertions should not cover whitespace characters (spaces, tabs) - only test actual content.
  - **Scope specificity requirement:** Assertions must use specific TextMate scope names from the grammar (e.g., `keyword.control.mermaid`, `variable`, `string.quoted.double.mermaid`, `punctuation.separator.comma.mermaid`). Do not use generic scopes like `source.mermaid` or built-in values - the goal is human-readable/parsable test output.
  - **Grammar-first approach:** Before writing tests, ensure the grammar (`syntaxes/diagrams/*.yaml`) defines proper capture groups with specific scope names. If a grammar element lacks captures for parameters (like commas, strings, variables), either update the grammar to add them or test only the elements that are captured.
  - **Assertions must never be `source.mermaid`.** Use the most specific scope available (e.g., `variable`, `string.quoted.double.mermaid`, `punctuation.separator.comma.mermaid`).
  - **Assertion lines for a single line under test can never overlap.** Each character position on the line can only be asserted by exactly one assertion line. Multiple assertion lines must cover contiguous, non-overlapping spans.
  - Use comments (`%%`) for assertion lines - these are ignored by Mermaid but read by the test framework.
  - Assertion lines must directly follow the line they are testing (no blank lines between).
- **Defining Scopes:** There are two ways to identify the expected scope for a token:
  1. **Arrow (`<-----`):** Used when the text to check is at the very beginning of the line. The number of dashes corresponds to the number of characters from the start of the line to check.
     ```mermaid
     graph TD
     %% <----- keyword.control.mermaid
     %%     ^^ entity.name.function.mermaid
     ```
  2. **Caret (`^^^^`):** Used to check specific spans of text. The carets must align vertically with the characters on the line above.
     ```mermaid
     ID-1
     %%^^^^ variable
     ```

**Example of complete scope coverage:**

```mermaid
graph TD
%% <----- keyword.control.mermaid
%%    ^^ entity.name.function.mermaid
    A[Start]
%%  ^ variable
%%   ^ punctuation.section.group.begin.mermaid
%%    ^^^^^ string.quoted.double.mermaid

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bpruitt-goddard/vscode-mermaid-syntax-highlight](https://github.com/bpruitt-goddard/vscode-mermaid-syntax-highlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
