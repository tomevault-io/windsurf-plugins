---
trigger: always_on
description: Temper reference: check
---



# Check: Stack-Aware Validation Pipeline

**Goal:** Run the project's full validation pipeline. Auto-detects stack and runs the right commands.

## Active Skills

- **Context Engineering** — load hierarchical context at stage start (rules → arch → source → errors, under 2K lines/task)
- **Temper Core** — stack detection, pack resolution, quality gates

## Execution

### Context Loading

This stage may run in two modes:
- **Standalone** (`/temper:check`) — runs in current context, handles its own gate
- **Agent subprocess** (from `/temper`) — starts with CLEAN context, no prior files needed

**Subprocess mode override:** When running as an Agent subprocess, do NOT show AskUserQuestion gates or clear context. Return the check summary to the orchestrator. The orchestrator handles all gate decisions and context transitions.

In both modes, the check methodology is identical.

Files to load at start:
1. `$CLAUDE_PLUGIN_ROOT/.claude-plugin/reference/check.md` (this file)
2. `.temper/specs/{feature}/review-context.json` (if exists — review findings for context)

### Step 1: Detect Stack

For stack detection order, apply the temper-core skill. Detection produces a stack identifier that determines which validation commands to run.

Stack-specific validation commands:

   pom.xml OR build.gradle → Java/Spring Boot
     compile: ./mvnw compile OR ./gradlew compileJava
     test:    ./mvnw test OR ./gradlew test
     build:   ./mvnw package OR ./gradlew build

   package.json → Node.js (check scripts section for commands)
     Read package.json scripts:
     test:  npm test (or whatever "test" script runs)
     build: npm run build (or whatever "build" script runs)
     lint:  npm run lint (if exists)
     type:  npx tsc --noEmit (if tsconfig.json exists)

   pyproject.toml OR setup.py → Python
     test:  pytest
     lint:  ruff check . (or flake8, pylint)
     type:  mypy . (if configured)
     build: python -m build

   go.mod → Go
     test:  go test ./...
     lint:  golangci-lint run
     build: go build ./...

   Cargo.toml → Rust
     test:  cargo test
     lint:  cargo clippy
     build: cargo build

Company preset OVERRIDES auto-detected commands.

### Step 2: Run Validation Levels (in order, stop on BLOCK-level failure)

NOTE: "Stop on failure" means halt the pipeline at the current level. Levels marked WARN continue to the next level. Only STOP/IMMEDIATELY/BLOCK results halt the pipeline.

```
Level 0: ENVIRONMENT
  Purpose: Verify not hitting production
  How: Check all .env* files (.env, .env.local, .env.production, etc.) for production indicators
       Verify DATABASE_URL and similar connection strings don't contain "production"
  If no .env files found: SKIP (not all projects use .env)
  If production detected: STOP IMMEDIATELY

Level 1: COMPILE/BUILD
  Purpose: Code compiles without errors
  Command: {detected compile command}
  On failure: STOP, show error output, suggest fix

Level 2: UNIT TESTS
  Purpose: All unit tests pass
  Command: {detected test command}
  On failure: STOP, show failing test names, suggest fix
  Report: tests run, passed, failed, duration

Level 3: INTEGRATION TESTS (if available)
  Purpose: Integration tests pass
  Command: {detected integration test command, if separate from unit}
  On failure: STOP, show failing tests
  If no integration tests configured: SKIP

Level 4: COVERAGE (if available)
  Purpose: Meets threshold
  Command: {detected coverage command}
  Threshold: from temper.config (default 80%) or company preset
  On failure: WARN (not block by default), show coverage %
  If no coverage tool configured: SKIP

Level 4.5: SCENARIO VERIFICATION (Live Execution)
  Purpose: Execute each Gherkin scenario's matching test individually, showing real pass/fail
  Confidence: [PROVEN] — mechanical test runner output
  Prerequisite: intent.md exists at .temper/specs/{spec}/intent.md
    If running standalone: resolve {spec} by listing .temper/specs/ directories and
    using the most recently modified one. If build-state.json exists, read spec from there.
    If no specs found → SKIP Level 4.5 entirely.
  Config: check.live-scenarios in temper.config (default: prompt)
    Valid values: prompt | always | never
    prompt → ask user whether to run live verification
    always → always run live verification
    never  → skip live verification, use heuristic analysis only (v3.0.0 behavior)
    Any other value → treated as "prompt" (safe default)
  How:
    STEP 1 — Extract scenarios:
      Read intent.md → extract all Gherkin scenarios (name + Given/When/Then)

    STEP 2 — Match scenarios to tests:
      For each scenario, find the matching test file:
      a. If MCP code-review-graph available: call query_graph_tool to find test
         by scenario name annotation → [PROVEN] match
      b. Fallback: grep test files for scenario name (snake_case or camelCase)
         → [HEURISTIC] match
      c. If no match found → UNMATCHED

    STEP 3 — Gate (prompt mode only):
      Show matched/unmatched counts. Ask user:
      "Run live verification for {N} matched scenarios? [Y/n]"
      If user declines → skip to heuristic-only analysis (v3.0.0 behavior)

    STEP 4 — Execute each matched test individually:
      For each matched scenario + test, run the test individually:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galando/temper](https://github.com/galando/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
