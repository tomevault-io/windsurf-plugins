---
trigger: always_on
description: GitHub Agentic Workflows is a Go-based GitHub CLI extension for writing agentic workflows in natural language using markdown files, running them as GitHub Actions.
---

# GitHub Agentic Workflows (gh-aw)

GitHub Agentic Workflows is a Go-based GitHub CLI extension for writing agentic workflows in natural language using markdown files, running them as GitHub Actions.

## Important Note: gh-aw vs GitHub Copilot CLI

**gh-aw** is a GitHub CLI extension (`gh aw`) that compiles markdown workflows into GitHub Actions. It is **not** the GitHub Copilot CLI (`copilot` command). While workflows can use the Copilot CLI as an AI engine, gh-aw itself is a separate tool for workflow management and compilation.

- Use `gh aw` commands (e.g., `gh aw compile`, `gh aw run`) to work with agentic workflows
- Use `/agent` in GitHub Copilot Chat to invoke the unified `agentic-workflows` custom agent (specify your intent: create/debug/update/upgrade)
- The `copilot` CLI command is only used internally within workflows when specified as the engine

## Important: Using Skills

**BE LAZY**: Skills in `skills/` provide detailed, specialized knowledge about specific topics. **Only reference skills when you actually need their specialized knowledge**. Do not load or reference skills preemptively.

**When to use skills:**
- You encounter a specific technical challenge that requires specialized knowledge
- You need detailed guidance on a particular aspect of the codebase (e.g., console rendering, error messages)
- You're working with a specific technology integration (e.g., GitHub MCP server, Copilot CLI)

**When NOT to use skills:**
- For general coding tasks that don't require specialized knowledge
- When the information is already available in this AGENTS.md file
- For simple, straightforward changes

**Available Skills Directory**: `skills/`

Each skill provides focused guidance on specific topics. Reference them only as needed rather than loading everything upfront.

## Skill Optimizer Configuration

The daily skill-optimizer workflow uses the committed config at:

- `.skill-optimizer/skill-optimizer.json`

When tuning optimizer behavior (models, maxTasks, maxIterations, pass-rate target), update this file directly in a PR so changes are reviewable.

## Critical Requirements

### ⚠️ MANDATORY: Always Create a Pull Request ⚠️

**🚨 IF YOU ARE THE GITHUB COPILOT CODING AGENT AND YOU HAVE FILE CHANGES, YOU MUST ALWAYS CREATE A PULL REQUEST - NO EXCEPTIONS.**

Use the **report_progress** tool to commit and push your changes. This will automatically create or update the pull request.

**Never leave file changes uncommitted.** Even for small or "obvious" changes, always use **report_progress** to push your work to a PR so it can be reviewed.

### ⚠️ MANDATORY PRE-COMMIT VALIDATION ⚠️

**🚨 BEFORE EVERY COMMIT - NO EXCEPTIONS:**

```bash
make agent-finish  # Runs build, test, recompile, fmt, lint
```

**Why this matters:**
- **CI WILL FAIL** if you skip this step - this is automatic and non-negotiable
- Unformatted code causes immediate CI failures that block all other work
- This has caused **5 CI failures in a single day** - don't be the 6th!
- The formatting check (`go fmt`) is strict and cannot be disabled

**If you're in a hurry** and `make agent-finish` takes too long, **at minimum run**:
```bash
make fmt         # Format Go, JavaScript, and JSON files
make test-unit   # Fast unit tests (~25s)
```

**After making Go code changes (*.go files):**
```bash
make fmt         # REQUIRED - formats Go code with go fmt
```

**After making workflow changes (*.md files):**
```bash
make recompile   # REQUIRED - recompile all workflow files after code changes
```

**After making JavaScript changes (*.cjs files):**
```bash
make fmt-cjs     # REQUIRED - ensures JavaScript is properly formatted
```

**NEVER ADD LOCK FILES TO .GITIGNORE** - `.lock.yml` files are compiled workflows that must be tracked.

**ALWAYS REBUILD AFTER SCHEMA CHANGES:**
```bash
make build       # Rebuild gh-aw after modifying JSON schemas in pkg/parser/schemas/
```
Schema files are embedded in the binary using `//go:embed` directives, so changes require rebuilding the binary.

**ALWAYS ADD BUILD TAGS TO TEST FILES:**

Every test file (`*_test.go`) **must** have a build tag at the very top of the file:

```go
//go:build !integration    // For unit tests (default)

//go:build integration     // For integration tests (files with "integration" in name)
```

**Rules:**
- Files with "integration" in the filename get `//go:build integration`
- All other test files get `//go:build !integration`
- The build tag must be the **first line** of the file, followed by an empty line

**To add build tags to all test files:**
```bash
./scripts/add-build-tags.sh
```

**ALWAYS RUN LINTERS AFTER ADDING TEST FILES:**

When adding new test files (`*_test.go`), the **unused** linter may catch helper functions that are defined but never called. Always run linters after creating test files to catch these issues early.

```bash
make lint        # Catches unused, testifylint, misspell, unconvert issues
```

**Common linting issues in test files:**

1. **unused**: Helper functions defined but never called
   - ❌ BAD: Defining `func hasInternalPrefix(key string) bool { ... }` but never using it
   - ✅ GOOD: Either use the function in tests or remove it


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [github/gh-aw](https://github.com/github/gh-aw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
