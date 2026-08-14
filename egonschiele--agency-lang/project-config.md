---
trigger: always_on
description: Agency is a domain-specific language for defining AI agent workflows. It compiles Agency code to executable TypeScript that calls OpenAI's structured output API.
---

# Agency Language

## Overview

Agency is a domain-specific language for defining AI agent workflows. It compiles Agency code to executable TypeScript that calls OpenAI's structured output API.

Please read the guide at docs/site/guide/ to get up to speed on the language.

NOTE! Most of the file paths you'll see in this CLAUDE.md are relative to the packages/agency-lang directory, as that is the main package and contains all the code for agency lang.

## Key Commands

```bash
make                    # build everything (ALWAYS use this when changing stdlib files)
pnpm test               # Run vitest in watch mode
pnpm test:run           # Run vitest once
pnpm run agency <file>  # Compile and run an .agency file
pnpm run agency test <file>  # Run a single Agency test
pnpm run agency test js <file>  # Run a single Agency js test
pnpm run compile <file> # Compile .agency to .ts
pnpm run ast <file>     # Parse .agency and print AST as JSON
pnpm run preprocess <file>     # Parse .agency, run preprocessor, and print the resulting AST as JSON
pnpm run fmt <file>     # Format .agency file using the AgencyGenerator
make fixtures           # Rebuild all integration test fixtures
pnpm run lint:structure # Run structural linter
```

## The full pipeline

`parse → SymbolTable.build → buildCompilationUnit → TypescriptPreprocessor → TypeScriptBuilder.build() → printTs()`

## Testing

See `docs/misc/TESTING.md` for the full testing guide.

Agency execution tests (`tests/agency/`) do NOT require LLM calls. They can test pure logic, interrupts, async calls, etc. without any LLM involvement. Use them for any runtime behavior test.

Agency-js tests (`tests/agency-js/`) are similar, but let you test how agency code interacts with js code.

Note that although agency and agency-js tests don't *require* LLM calls, they can support them if needed. Don't make any extra LLM calls because they are slow and expensive, but if you are writing a test and you *need* to make an LLM call, please feel free to make one.

IMPORTANT! When you run tests, save the output to a file so that if the tests fail, you don't need to rerun them to see what failed. The tests in this repo are very expensive and slow to rerun, so if you keep rerunning tests to see what failed, you're going to waste a lot of time. Just run the test and save the output in a file once so you can examine the output at your leisure.

Note: Do not run the agency test suite locally. It takes a long time to run. When you create a PR, CI will run those tests for you. In the meantime, you're welcome to run specific agency tests locally if they are relevant to your change, but don't run the full test suite.

## Documentation

Standard library reference documentation is generated from Agency source using `agency doc`. When adding or changing stdlib APIs, write module-level doc comments, function doc comments, and docstrings in the `.agency` source files instead of hand-editing generated `docs/site/stdlib/*.md` pages. Docstrings also become tool descriptions for LLM-callable functions, so keep them accurate and user-facing. See `docs/site/cli/doc.md` for the `agency doc` conventions.

## CRITICAL: Handlers are safety infrastructure
Handlers (`handle` blocks) are a crucial part of what makes Agency safe. They must NEVER be accidentally skipped or left unregistered. Any feature that affects execution flow (rewind, interrupts, checkpoints, state restoration) must ensure handlers are correctly registered and invoked. If there is any risk of a handler being skipped, treat it as a critical issue and flag it immediately. Handlers are registered on `__ctx.handlers` via `pushHandler()` in the generated code and are NOT serialized as part of checkpoint state — be aware of this when working on state restoration features.

## VERY IMPORTANT: Agency syntax rules
When writing Agency code (in plans, specs, tests, or examples), you MUST use the correct syntax. Verify against `docs/site/guide/basic-syntax.md` and existing test fixtures when unsure.

**Correct syntax:**
- Functions use `def`, curly braces, and optional `: ReturnType` after params: `def foo(x: number): string { ... }`
- Nodes use `node`, parentheses for params, and curly braces: `node main() { ... }`
- `if`, `while`, and `for` statements REQUIRE parentheses around the condition AND curly braces for the body: `if (x > 5) { ... }`
- Variables must be declared with `let` or `const` before use. Bare assignment (`x = 5`) is NOT allowed without a prior declaration.
- `for` loops use `in`: `for (item in items) { ... }`

**Common mistakes to NEVER make:**
- `function foo() -> ReturnType:` — WRONG. Use `def foo(): ReturnType { ... }`
- `node main -> end:` — WRONG. Use `node main() { ... }`
- `if condition:` / `if condition {` — WRONG. Use `if (condition) { ... }`
- `result = foo()` without `let`/`const` — WRONG unless already declared.
- Using Python-style colon+indentation for blocks — WRONG. Always use `{ ... }`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [egonSchiele/agency-lang](https://github.com/egonSchiele/agency-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
