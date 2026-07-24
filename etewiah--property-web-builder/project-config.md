---
trigger: always_on
description: This file contains instructions for Claude Code instances working on this project.
---

# Claude Code Instructions for PropertyWebBuilder

This file contains instructions for Claude Code instances working on this project.

## Critical Rules

> **🚨 STOP! BEFORE ANY GIT COMMIT 🚨**
>
> You MUST ask the user "Would you like me to commit these changes?" and wait for explicit confirmation BEFORE running `git commit`. This is NON-NEGOTIABLE. Never auto-commit.

### Git Commit Safety

**NEVER commit changes you did not make in this session.**

**ALWAYS ask for user confirmation before committing.** Do not commit automatically. Show the user:
1. The files that will be committed
2. The proposed commit message
3. Wait for explicit approval (e.g., "yes", "ok", "commit it") before running `git commit`

**If you commit without asking first, you have violated the most important rule in this file.**

Before committing:
1. Run `git status` to see ALL modified and untracked files
2. Identify which files YOU modified vs files that were already modified before your session
3. Only stage files YOU explicitly created or modified
4. If you see untracked files or modifications you didn't make, DO NOT add them

When staging files:
- Use specific file paths: `git add path/to/file.rb` instead of `git add .` or `git add -A`
- After staging, verify with `git diff --cached --stat` that only your changes are included
- If unsure, ask the user before committing

Common mistakes to avoid:
- Using `git add .` which adds ALL changes including other sessions' work
- Using `git add folder/` when other files in that folder were modified by someone else
- Not checking `git status` before committing

### No Direct Database Changes

**NEVER make direct database changes to fix issues.** All fixes must be made in code (seed files, migrations, rake tasks, etc.) that can be committed to git.

- If you modify database records directly (e.g., via `rails runner`, `rails console`, or any SQL), you have NOT fixed the issue
- Always find and edit the source files (seed YAML files, migrations, etc.) that populate the database
- If the only solution appears to be a direct database edit, you MUST explicitly ask for permission first and explain why a code-based solution isn't possible
- Never claim an issue is "fixed" when only database records were changed - such changes are lost on reseed/reset

### Bug Fixing and Test Coverage

> **🔴 MANDATORY: Every bug fix MUST include test coverage analysis**

**After fixing any bug or error, you MUST:**
1. Analyze why the bug wasn't caught by existing tests
2. Add a test that would have caught it
3. Search for similar issues elsewhere in the codebase

This is NOT optional. Do not consider a bug "fixed" until you've completed all three steps.

#### Step 1: Analyze the Gap

Ask yourself: **"Why was this not caught in a test?"** Common reasons:
- No test exists for the affected code path
- Tests mock at too high a level and don't exercise the actual code
- Tests don't cover the specific configuration/integration being used
- Hardcoded values bypass configurable settings
- Async jobs are enqueued but not executed in tests
- Autoloading behaves differently in test vs runtime
- Edge cases or error handling paths aren't covered

#### Step 2: Add a Test

Create a test that would have caught this bug. The test should:
- Fail without your fix (verify by mentally reviewing)
- Pass with your fix
- Be specific enough to catch regressions
- Test the actual integration, not just mocked behavior

#### Step 3: Check for Similar Issues

**Use grep/search to find similar patterns** that might have the same problem:
- If you fixed a missing `super()` argument, check other subclasses
- If you fixed a hardcoded value ignoring settings, search for similar hardcoding
- If you fixed an autoloading issue, check similar module structures
- If you fixed a multi-tenant scope issue, check similar queries

**Example: Search command**
```bash
# If you fixed "model: DEFAULT_MODEL" ignoring integration settings:
grep -r "model: DEFAULT_MODEL" app/services/
```

#### Examples

**Example 1: Missing parent initialization**
```
1. User reports: "AI is not configured" error
2. Fix: Pass `website` to parent class in ScriptGenerator
3. Ask: "Why wasn't this caught?" → No tests for ScriptGenerator existed
4. Add: spec/services/video/script_generator_spec.rb with 27 tests
5. Check: grep for "super()" in other AI service subclasses
```

**Example 2: Hardcoded values ignoring configuration**
```
1. User reports: OpenRouter configured but still uses Anthropic
2. Fix: Remove hardcoded `model: DEFAULT_MODEL` from call_llm methods
3. Ask: "Why wasn't this caught?" → Tests mocked the chat() method
4. Add: Test that verifies integration's model setting is respected
5. Check: grep "model: DEFAULT_MODEL" found 2 more services with same issue
```

## Documentation Guidelines

**IMPORTANT: Never create documentation files at the project root.**

All documentation must be placed in the `docs/` folder structure:

- `docs/` - General project documentation
- `docs/architecture/` - Architecture decisions and system design
- `docs/seeding/` - Seed data and seed packs documentation
- `docs/multi_tenancy/` - Multi-tenancy related documentation
- `docs/claude_thoughts/` - Claude's research, analysis, and exploratory notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [etewiah/property_web_builder](https://github.com/etewiah/property_web_builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
