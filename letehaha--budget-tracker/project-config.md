---
trigger: always_on
description: - **Default to caveman mode** for all user-facing text in this project. Follow the conventions in `.claude/skills/caveman/SKILL.md` (or the `caveman` skill description) — drop articles/filler/pleasantries, keep technical accuracy. Apply to chat replies, status updates, and end-of-turn summaries. Do NOT apply to code, comments, commit messages, PR descriptions, or file content.
---

# Project Rules & Conventions

## Communication Style

- **Default to caveman mode** for all user-facing text in this project. Follow the conventions in `.claude/skills/caveman/SKILL.md` (or the `caveman` skill description) — drop articles/filler/pleasantries, keep technical accuracy. Apply to chat replies, status updates, and end-of-turn summaries. Do NOT apply to code, comments, commit messages, PR descriptions, or file content.
- Switch to normal English only when the user explicitly asks (e.g. "normal mode", "stop caveman", "full sentences").

- **Backend work**: Read `.claude/docs/backend-conventions.md` before writing backend code
- **Frontend work**: Read `.claude/skills/frontend-rules/SKILL.md` before writing frontend code

## Testing Patterns

**CRITICAL**: E2E tests in this project NEVER call services directly. They ONLY make HTTP endpoint calls through the test helpers.

Examples:

- ❌ WRONG: `await syncHistoricalPrices(securityId)`
- ✅ CORRECT: `await helpers.createHolding({ payload: { portfolioId, securityId } })` (which triggers sync internally)
- ❌ WRONG: `await someService.doSomething()`
- ✅ CORRECT: `await helpers.makeRequestToEndpoint()`

Always test through the actual API endpoints to ensure full integration testing.

**CRITICAL: E2E Tests Required for New Backend Endpoints**

- Every new backend endpoint (route + controller + service) **MUST** include an e2e test before the work is considered complete.
- **Auto-trigger**: After implementing a new endpoint, automatically write e2e tests as the next step — don't wait to be asked.
- Minimum coverage: **happy path**, **empty state**, and at least one **error case**.
- Follow the `e2e-test-creator` skill conventions (`.claude/skills/e2e-test-creator/SKILL.md`) for structure and patterns.
- Run the tests automatically after writing them — do not wait for user confirmation.

**Bug Fix Workflow: Test-First Approach**

- When a bug is reported, do **NOT** start by trying to fix it.
- **First**, write a test that reproduces the bug (the test should fail) if it's suitable. For backend use e2e tests, for frontend if it's a util/composable write unit-test.
- **Then**, use subagents to fix the bug and prove it with a passing test.

**CRITICAL: Running Tests**

- **Run e2e tests automatically** after implementing changes that affect backend logic (new endpoints, bug fixes, refactors). Do not wait for user confirmation — just run them.
- **NEVER** use `npx jest` directly. Always use the npm scripts.
- **ALWAYS** use the `test-runner` subagent to run tests. The main agent must NEVER run tests directly.
- Backend e2e tests: `npm run test:e2e` from `packages/backend/`
- To run a specific test file: `npm run test:e2e -- --testPathPattern='<pattern>'`
- Example: `npm run test:e2e -- --testPathPattern='subscriptions/subscriptions.e2e'`
- **NEVER** run e2e tests in parallel (no concurrent test-runner agents for e2e). The Docker-based test environment does not support parallel runs. To test multiple files, combine them in a single `--testPathPattern` regex.
- Example (multiple files): `npm run test:e2e -- --testPathPattern='subscriptions/(subscriptions|matching-disambiguation).e2e'`

Other instructions:

1. File names should always be in kebab-case
2. All functions should _always_ use object-like params.
   - Never: function(arg1, arg2, arg3)
   - Always: function({ arg1, arg2, arg3 })
3. When planning the implementation don't limit yourself to 3-4 questions and 1 round.
   Ask as many questions with as many rounds as needed to collect all important information
4. Use this map of suagents for different tasks:
   - running any unit or e2e tests – use test-runner
   - running any linter – use linter
     – planning doing any websearch – use websearch
     – if asked to do any code review – use code-change-reviewer
5. **Tool Selection for Code Search:**

   **Use ast-grep when:**
   - Finding code patterns by **structure** (not just text):
     - Function calls with specific argument patterns
     - Class/interface definitions matching criteria
     - Conditionals without certain clauses (e.g., `if` without `else`)
   - **i18n migrations**: Distinguishing user-facing text from technical strings
     - Finds `<h1>Welcome</h1>` (needs i18n) vs `console.log('debug')` (doesn't)
     - Locates strings not wrapped in `$t()` or `t()`
   - **Refactoring**: Renaming/restructuring based on syntax tree
   - **Context-aware searches**: Code in specific syntactic positions

   **Use grep when:**
   - Simple literal searches (variable names, specific constants)
   - Cross-file-type searches (markdown, JSON, configs, code)
   - Exploratory searches (don't know exact structure yet)
   - Finding comments, documentation, string literals

   **Examples:**

   ```bash
   # ❌ Use ast-grep instead
   grep -r "useState" --include="*.tsx"  # Finds in comments, strings

   # ✅ Use ast-grep for structural search
   ast-grep --pattern 'useState($$$)' src/**/*.tsx


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [letehaha/budget-tracker](https://github.com/letehaha/budget-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
