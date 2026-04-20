---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.
always use the babysitter skill to fulfill USER requests (but not agent requests)

## Commands by Package

### Root (npm workspaces)

```bash
npm install                                    # Install all workspace dependencies
npm run build:sdk                              # Build SDK
npm run test:sdk                               # Run SDK tests
npm run test:e2e:docker                        # E2E tests (vitest, config: e2e-tests/docker/vitest.config.ts)
npm run verify:metadata                        # Check README/package metadata
```

### SDK (`packages/sdk` / `@a5c-ai/babysitter-sdk`)

```bash
npm run build --workspace=@a5c-ai/babysitter-sdk    # tsc -> dist/
npm run clean --workspace=@a5c-ai/babysitter-sdk    # rimraf dist
npm run lint --workspace=@a5c-ai/babysitter-sdk     # eslint "src/**/*.ts" --max-warnings=0
npm run lint --workspace=@a5c-ai/babysitter-sdk -- --fix  # ESLint autofix
npm run test --workspace=@a5c-ai/babysitter-sdk     # vitest run (all tests)
npm run test:watch --workspace=@a5c-ai/babysitter-sdk  # vitest watch mode (script name: test:watch)
cd packages/sdk && npx vitest run src/runtime/__tests__/someFile.test.ts  # Single test file
cd packages/sdk && npm run smoke:cli                # CLI smoke test
babysitter mcp:serve [--json]                        # Start MCP server over stdio
```

### Babysitter CLI Reference

```bash
# Run Management
babysitter run:create --process-id <id> --entry <path#export> [--runs-dir <dir>] [--inputs <file>] [--run-id <id>] [--process-revision <rev>] [--request <id>] [--prompt <text>] [--harness <name>] [--session-id <id>] [--plugin-root <dir>] [--non-interactive] [--json] [--dry-run]
babysitter run:status <runDir> [--runs-dir <dir>] [--json]
babysitter run:events <runDir> [--runs-dir <dir>] [--json] [--limit <n>] [--reverse] [--filter-type <type>]
babysitter run:rebuild-state <runDir> [--runs-dir <dir>] [--json] [--dry-run]
babysitter run:repair-journal <runDir> [--runs-dir <dir>] [--json] [--dry-run]
babysitter run:iterate <runDir> [--runs-dir <dir>] [--json] [--verbose] [--iteration <n>]
babysitter run:execute-tasks <runDir> [--runs-dir <dir>] [--json] [--verbose] [--dry-run] [--max-tasks <n>] [--kind <kind>] [--timeout <ms>]

# Task Management
babysitter task:post <runDir> <effectId> --status <ok|error> [--runs-dir <dir>] [--json] [--dry-run] [--value <file>] [--value-inline <json>] [--error <file>] [--stdout-ref <ref>] [--stderr-ref <ref>] [--stdout-file <file>] [--stderr-file <file>] [--started-at <iso8601>] [--finished-at <iso8601>] [--metadata <file>] [--invocation-key <key>]
babysitter task:list <runDir> [--runs-dir <dir>] [--pending] [--kind <kind>] [--json]
babysitter task:show <runDir> <effectId> [--runs-dir <dir>] [--json]

# Session Management
babysitter session:init --session-id <id> --state-dir <dir> [--max-iterations <n>] [--run-id <id>] [--prompt <text>] [--json]
babysitter session:associate --session-id <id> --state-dir <dir> --run-id <id> [--force] [--runs-dir <dir>] [--json]
babysitter session:resume --session-id <id> --state-dir <dir> --run-id <id> [--max-iterations <n>] [--runs-dir <dir>] [--json]
babysitter session:state --session-id <id> --state-dir <dir> [--json]
babysitter session:update --session-id <id> --state-dir <dir> [--iteration <n>] [--last-iteration-at <iso8601>] [--iteration-times <csv>] [--delete] [--json]
babysitter session:check-iteration --session-id <id> --state-dir <dir> [--json]
babysitter session:last-message --transcript-path <file> [--json]
babysitter session:iteration-message --iteration <n> [--run-id <id>] [--runs-dir <dir>] [--plugin-root <dir>] [--json]

# Skill Discovery
babysitter skill:discover --plugin-root <dir> [--run-id <id>] [--cache-ttl <seconds>] [--runs-dir <dir>] [--include-remote] [--summary-only] [--process-path <path>] [--json]
babysitter skill:fetch-remote --source-type <github|well-known> --url <url> [--json]

# Harness Management
babysitter harness:discover [--json]
babysitter harness:list [--json]                      # Alias for harness:discover
babysitter harness:invoke <name> --prompt <text> [--workspace <dir>] [--model <model>] [--timeout <ms>] [--json]
babysitter harness:create-run [--prompt <text>] [--harness <name>] [--process <path>] [--workspace <dir>] [--model <model>] [--max-iterations <n>] [--runs-dir <dir>] [--interactive|--no-interactive|--non-interactive] [--json] [--verbose]
babysitter harness:call [...]                         # Alias for harness:create-run
babysitter harness:yolo [...]                         # Alias for harness:create-run --non-interactive
babysitter harness:plan [...]                         # Alias for harness:create-run, stops after Phase 1
babysitter harness:forever [...]                      # Alias for harness:create-run, infinite loop process
babysitter harness:resume-run [--run-id <id>] [--runs-dir <dir>] [--harness <name>] [--workspace <dir>] [--model <model>] [--max-iterations <n>] [--interactive|--no-interactive] [--json] [--verbose]
babysitter harness:resume [...]                       # Alias for harness:resume-run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a5c-ai/babysitter](https://github.com/a5c-ai/babysitter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
