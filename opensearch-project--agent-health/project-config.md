---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENT.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

AgentEval is an evaluation framework for Root Cause Analysis (RCA) agents using "Golden Path" trajectory comparison. An LLM Judge (AWS Bedrock Claude) evaluates agent actions against expected trajectories to score performance.

## Build Commands

```bash
# Install dependencies
npm install

# Development - run both servers simultaneously
npm run dev           # Frontend at http://localhost:4000
npm run dev:server    # Backend at http://localhost:4001

# Build and test
npm run build         # TypeScript check + Vite build
npm test              # Run Jest tests
npm test -- --watch   # Watch mode
npm test -- path/to/file.test.ts  # Single test file
```

## Before Committing

Always run tests before committing changes:

```bash
npm test              # Run all tests - must pass before pushing
```

Update `CHANGELOG.md` under `## [Unreleased]` with your changes:
- `### Added` - New features
- `### Changed` - Changes to existing functionality
- `### Fixed` - Bug fixes
- `### Security` - Security fixes

## Before Raising a PR

All PRs are validated by CI. Fix these locally before pushing to avoid failed checks:

```bash
# 1. Build and test
npm run build:all && npm run test:all

# 2. Security scan - no high/critical vulnerabilities
npm audit --audit-level=high

# 3. Verify DCO signoff on all commits
git log origin/main..HEAD | grep "Signed-off-by"
# If missing, fix with: git rebase origin/main --signoff

# 4. Verify changelog is updated
grep -A5 "## \[Unreleased\]" CHANGELOG.md
```

**Pre-PR Checklist:**
- [ ] All commits have DCO signoff (`git commit -s`)
- [ ] `CHANGELOG.md` updated under `## [Unreleased]` with PR link
- [ ] `npm run build:all` succeeds
- [ ] `npm run test:all` passes
- [ ] `npm audit --audit-level=high` reports no vulnerabilities
- [ ] New source files have SPDX license headers

### CLI: Import Test Cases from JSON

The `benchmark` command supports importing test cases from a JSON file via `-f` / `--file`:

```bash
# Import and benchmark in one step
npx @opensearch-project/agent-health benchmark -f ./test-cases.json -a my-agent

# With a custom benchmark name
npx @opensearch-project/agent-health benchmark -f ./test-cases.json -n "My Benchmark" -a my-agent

# Export produces import-compatible JSON (round-trip support)
npx @opensearch-project/agent-health export -b my-benchmark -o test-cases.json
```

The JSON file must be an array of test case objects with required fields: `name`, `category`, `difficulty`, `initialPrompt`, `expectedOutcomes`.

## Environment Setup

Copy `.env.example` to `.env`. Key variables:

- `PORT` - Backend port (default: 4001)
- `MLCOMMONS_ENDPOINT` - ML-Commons agent streaming endpoint
- `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` - Bedrock credentials
- `OPENSEARCH_STORAGE_*` - OpenSearch cluster for persistence
- `OPENSEARCH_LOGS_*` - OpenSearch cluster for logs/traces

## Architecture

> **Full documentation:** See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for detailed architecture patterns.
> **Performance optimization:** See [docs/PERFORMANCE.md](docs/PERFORMANCE.md) for performance optimizations in the Benchmark Runs Overview page.

### Two-Server Architecture

- **Frontend (Vite + React)**: Port 4000 (development) - UI for running evaluations
- **Backend (Express)**: Port 4001 - Proxy for Bedrock API calls (browser cannot call Bedrock directly)
- **Production**: Port 4001 serves both frontend and backend

### Core Data Flow

```
User selects agent + test case
    → Agent streams AG-UI events via SSE
    → AGUIToTrajectoryConverter builds TrajectoryStep[]
    → Backend calls Bedrock Judge for evaluation
    → Report stored (localStorage or OpenSearch)
```

### Services Layer (`services/`)

| Directory     | Purpose                                                                                                                                |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `agent/`      | AG-UI protocol handling: SSE streaming (`sseStream.ts`), event conversion (`aguiConverter.ts`), payload building (`payloadBuilder.ts`) |
| `evaluation/` | Orchestrates evaluation runs (`index.ts`), Bedrock judge client with retry (`bedrockJudge.ts`)                                         |
| `storage/`    | Async storage with OpenSearch backend (`asyncRunStorage.ts`, `asyncTestCaseStorage.ts`, `asyncExperimentStorage.ts`)                   |
| `traces/`     | Trace transformations: Flow view, Timeline view, comparison alignment, tool similarity grouping                                        |
| `opensearch/` | Log fetching from OpenSearch clusters                                                                                                  |

### Key Types (`types/index.ts`)

- `TestCase` - Use case definition with versioned content and expected trajectory
- `TestCaseRun` (alias: `EvaluationReport`) - Result of running a test case
- `TrajectoryStep` - Single step in agent execution (tool_result, assistant, thinking, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensearch-project/agent-health](https://github.com/opensearch-project/agent-health) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
