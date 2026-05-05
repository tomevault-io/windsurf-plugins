---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Build
go build -o ckb ./cmd/ckb

# Run all tests
go test ./...

# Run tests for a specific package
go test ./internal/query/...

# Run a single test
go test ./internal/query/... -run TestSearchSymbols

# Run tests with verbose output
go test -v ./...

# Run with coverage
go test -cover ./...

# Lint (if golangci-lint installed)
golangci-lint run
```

## Running the Application

```bash
# Initialize CKB in a project directory
./ckb init

# Generate SCIP index (auto-detects language)
./ckb index

# Check system status
./ckb status

# Run diagnostics
./ckb doctor

# Start HTTP API server
./ckb serve --port 8080

# Start MCP server (for AI tool integration)
./ckb mcp

# Start A2A protocol server (for agent-to-agent communication)
./ckb a2a --port 8081

# Find exported symbols not reachable from entrypoints
./ckb unwired
./ckb unwired --scope internal/cost,internal/multisampling
./ckb unwired --format json

# Run PR review (21 quality checks)
./ckb review
./ckb review --base=develop --format=markdown
./ckb review --checks=breaking,secrets,health --ci

# Run compliance audit (131 checks across 20 regulatory frameworks)
./ckb audit compliance --framework=gdpr
./ckb audit compliance --framework=gdpr,iso27001,owasp-asvs
./ckb audit compliance --framework=all --min-confidence=0.7 --format=sarif
./ckb audit compliance --framework=pci-dss,hipaa --ci --fail-on=error
./ckb audit compliance --framework=iec61508 --sil-level=3

# Auto-configure AI tool integration (interactive)
./ckb setup

# Configure for specific AI tools
./ckb setup --tool=claude-code
./ckb setup --tool=cursor
./ckb setup --tool=grok
./ckb setup --tool=vscode
```

## npm Distribution (v7.0)

CKB is also available via npm:

```bash
# Install globally
npm install -g @tastehub/ckb

# Or run directly with npx
npx @tastehub/ckb init
npx @tastehub/ckb setup
```

## MCP Integration

CKB provides 80+ code intelligence tools via MCP. Supports Claude Code, Cursor, Windsurf, VS Code, OpenCode, Grok, and Claude Desktop.

```bash
# Auto-configure (interactive)
ckb setup

# Configure for specific tool
ckb setup --tool=claude-code
ckb setup --tool=cursor --global

# Or manually add to .mcp.json
claude mcp add ckb -- npx @tastehub/ckb mcp
```

`ckb setup --tool=claude-code` also installs the `/ckb-review` and `/ckb-audit` slash commands for Claude Code, which orchestrate CKB's structural analysis with LLM semantic review.

### Key MCP Tools

**Navigation:** `searchSymbols`, `getSymbol`, `findReferences`, `getCallGraph`, `traceUsage`, `listEntrypoints`

**Understanding:** `explainSymbol`, `explainFile`, `explainPath`, `getArchitecture`, `listKeyConcepts`

**Impact & Risk:** `analyzeImpact`, `justifySymbol`, `getHotspots`, `summarizeDiff`, `summarizePr`

**Ownership:** `getOwnership`, `getModuleResponsibilities`, `getOwnershipDrift`

**Decisions:** `recordDecision`, `getDecisions`, `annotateModule`

**Federation (v6.2+):** `listFederations`, `federationSearchModules`, `federationGetHotspots`

**Contracts (v6.3):** `listContracts`, `analyzeContractImpact`, `getContractDependencies`

**Telemetry (v6.4):** `getTelemetryStatus`, `getObservedUsage`, `findDeadCodeCandidates`

**Intelligence (v6.5):** `explainOrigin`, `analyzeCoupling`, `exportForLLM`, `auditRisk`

**Code Analysis (v7.6):** `findDeadCode` (static dead code detection), `findUnwiredModules` (entrypoint reachability — detects "built but never plugged in" modules), `getAffectedTests`, `compareAPI`

**Compound Operations (v8.0):** `explore`, `understand`, `prepareChange`, `batchGet`, `batchSearch`

**Streaming (v8.0):** `findReferences` and `searchSymbols` support SSE streaming with `stream: true`

**Index Management (v8.0):** `reindex` (trigger index refresh), enhanced `getStatus` with health tiers

**PR Review (v8.2):** `reviewPR` — unified review with 21 quality checks (breaking, secrets, tests, complexity, health, coupling, hotspots, risk, critical-path, traceability, independence, generated, classify, split, dead-code, unwired, test-gaps, blast-radius, comment-drift, format-consistency, bug-patterns); optional `--llm` flag for Claude-powered narrative

## A2A Integration

CKB also supports the A2A (Agent-to-Agent) protocol v0.3, enabling agent-to-agent communication over HTTP. All 80+ MCP tools are exposed as A2A skills.

```bash
# Start the A2A server
ckb a2a --port 8081

# Agent card (discovery)
curl http://localhost:8081/.well-known/agent-card.json

# Health with index freshness
curl http://localhost:8081/health

# Invoke a skill
curl -X POST http://localhost:8081/message:send \
  -d '{"message":{"role":"ROLE_USER","parts":[{"text":"{\"skill\":\"getStatus\",\"params\":{}}"}]}}'
```

## Architecture Overview

CKB is a code intelligence orchestration layer with four interfaces (CLI, HTTP API, MCP, A2A) that all flow through a central query engine.

### Layer Structure

```
Interfaces (cmd/ckb/, internal/api/, internal/mcp/, internal/a2a/)
    ↓
Query Engine (internal/query/)
    ↓
Backend Orchestrator (internal/backends/orchestrator.go)
    ↓
Backends: SCIP, LSP, Git (internal/backends/{scip,lsp,git}/)
    ↓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SimplyLiz/CodeMCP](https://github.com/SimplyLiz/CodeMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
