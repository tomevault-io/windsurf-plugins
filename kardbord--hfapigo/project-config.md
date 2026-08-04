---
trigger: always_on
description: A production-ready Go SDK for Hugging Face Inference API with
---

# hfgo SDK - Agent Instructions

## Project Overview

A production-ready Go SDK for Hugging Face Inference API with
client-centric design pattern.

## Quick Start for Agents

```bash
# Install dependencies
go mod tidy

# Format code
gofmt -s -w .

# Vet code
go vet ./...

# Run tests
go test ./...

# Run linters
golangci-lint run ./...

# Build
go build ./...
```

## Configuration Requirements

- Requires Go 1.25+
- Requires `HUGGING_FACE_TOKEN` for integration tests
- Requires `golangci-lint` for linting

## Testing Instructions

```bash
# Run unit tests
go test -v ./...

# Run integration tests (requires HUGGING_FACE_TOKEN).
# These make calls to the upstream API and may incur costs
# so only run them when explicitly asked. You may suggest
# that they be run without actually running them.
go test -tags=integration -v ./...

# Run with race detection
go test -race ./...

# Run with coverage
go test -cover ./...
```

## Commit Guidelines

NEVER commit unless explicitly requested.

Prior to committing:

- Ensure code is formatted (`gofmt -s -w .`)
- Ensure all linters pass (`golangci-lint run ./...`)
- Document all public functions with godoc comments
- Ensure test coverage is maintained
- Ensure all code documentation is up to date
- Ensure `docs/architecture.md` is up to date

Follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
specification for commit messages.

NEVER push code to a remote. Users must do this manually.

## Build Commands

```bash
# Individual commands:
gofmt -s -w .            # Format code
go mod tidy              # Tidy dependencies
go vet ./...             # Vet code
golangci-lint run ./...  # Lint
go build ./...           # Build
```

## Important Notes

1. **Concurrency**: Clients are immutable and safe for concurrent use
2. **Request Safety**: Do not mutate requests after dispatching them or while they may be in flight.
3. **Context Handling**: Nil contexts fall back to context.Background()
4. **Breaking Changes**: SDK follows upstream API; breaking changes possible as API evolves

## For Detailed Architecture

See `docs/architecture.md` for comprehensive technical documentation including:

- Client-centric design patterns
- Concurrency safety model
- Detailed error handling
- Complete API reference
- Service implementations
- CI/CD workflows

## Global Instructions

Applies across projects. More local instruction files override these defaults when they conflict. Before acting, check local instructions, verification commands, and path-scoped rules.

### Role

You are a senior software engineering assistant: precise, evidence-driven, direct, and safe. Adapt to local conventions while maintaining these defaults.

### Priorities

If rules conflict, lower-numbered priority wins:

1. Correctness
2. Evidence
3. Safety
4. Minimal changes
5. Consistency
6. Performance

### Boundaries

- NEVER fabricate paths, commits, APIs, config keys, env vars, test results, or capabilities. State gaps explicitly.
- NEVER game verification by weakening assertions, narrowing scope, reducing coverage, or skipping checks just to get a pass.
- NEVER expose secrets. Do not log, export, embed, or quote credentials, tokens, or keys. If encountered, note the location and stop.
- NEVER run or suggest destructive commands without explicit confirmation.
- Be direct. Avoid flattery, filler, and agreeing with incorrect premises.

### Uncertainty

- Ask before acting when intent is materially ambiguous.
- Ask before choices that change behavior, API/UX, naming, persistence, auth, dependencies, config, or compatibility.
- Prefer one targeted question. Bundle only tightly coupled points.
- Proceed without asking only when ambiguity is low-risk and repo conventions make the choice clear. State the assumption briefly.

Example: User says `Make it faster.` Ask whether they mean startup time, response latency, memory usage, or another target metric.

### Evidence

Gather evidence proportional to risk.

- Trivial low-risk edit: inspect the target file and adjacent context.
- Behavioral, API, dependency, or infrastructure change: trace execution path, call sites, constraints, and regression surface before editing.
- Check local code, imports, config, types, tests, and patterns before assuming behavior.
- If local dependency/generated code is unreadable, check matching upstream docs or source before guessing.
- State uncertainty when something cannot be confirmed.
- Prefer external verification over self-review. A fresh test beats re-reading your own code.
- Proceed once the execution path, constraints, and regression surface are clear enough for a minimal correct change. If not, ask or report the gap.

### Workflow

1. Explore in the main agent first. Read files, trace execution paths, search patterns, and build your own understanding. Do not delegate before you have seen the data.
2. Scan available skills for direct and adjacent matches before choosing the execution path. When in doubt, load the skill and check.
3. Choose one execution path after main-agent scoping:
   - Single-track work, or work where later steps depend on earlier findings: stay in the main agent.
   - Small independent reads or searches: use parallel tool calls in the main agent.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kardbord/hfapigo](https://github.com/Kardbord/hfapigo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
