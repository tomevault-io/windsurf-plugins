---
trigger: always_on
description: Warren is an AI agent and Slack-based security alert management tool that processes security alerts using LLM (Gemini) and manages incident response through Slack.
---

# CLAUDE.md

Warren is an AI agent and Slack-based security alert management tool that processes security alerts using LLM (Gemini) and manages incident response through Slack.

## Common Development Commands

### Building and Testing
- `go test ./...` - Run all tests
- `go test ./pkg/path/to/package` - Run tests for specific package
- `task` - Run default tasks (mock generation and GraphQL)
- `task mock` (alias: `task m`) - Generate all mock files
- `task graphql` - Generate GraphQL code from schema

### Frontend Development
- `cd frontend && pnpm install` - Install frontend dependencies
- `pnpm run dev` - Start development server
- `pnpm run build` - Build frontend for production
- `pnpm run codegen` - Generate GraphQL types from schema

### Code Generation
- `go tool moq` - Generate mocks (handled by task commands)
- `go tool gqlgen generate` - Generate GraphQL resolvers and types

## Architecture Overview

The application follows Domain-Driven Design (DDD) with clean architecture:

- `pkg/domain/` - Domain layer with business logic, interfaces, and models
- `pkg/service/` - Application services implementing business operations
- `pkg/controller/` - Interface adapters (HTTP, GraphQL, Slack)
- `pkg/adapter/` - Infrastructure adapters (storage, external APIs)
- `pkg/repository/` - Data persistence implementations
- `pkg/usecase/` - Application use cases orchestrating domain operations

### Alert Processing Pipeline
Pipeline stages in `pkg/usecase/alert_pipeline.go`:
1. **Ingest Policy Evaluation** - Transform raw alert data into Alert objects
2. **Tag Conversion** - Convert tag names to tag IDs
3. **Metadata Generation** - Fill missing titles/descriptions using LLM
4. **Enrich Policy Evaluation** - Execute enrichment tasks (query/agent)
5. **Triage Policy Evaluation** - Apply final metadata and determine publish type

### Application Modes
`serve` (HTTP/Slack/GraphQL), `run` (CLI), `chat` (interactive), `tool` (utilities), `test` (testing)

## Development Rules

### Implementation Completeness
- **NEVER leave incomplete implementations, TODOs, or placeholder code**
- **NEVER skip implementation because it's complex or lengthy**
- **ALWAYS complete the full implementation in one go**
- If a task seems too complex, break it down into smaller steps, but complete ALL steps
- Complexity is not an excuse - implement everything thoroughly
- Long code is acceptable - incomplete code is NOT

### Multi-Instance Safety (Stateless Design)
- **Warren is designed to run as multiple concurrent instances** (horizontal scaling). Any design that assumes single-instance will break in production
- **NEVER hold cross-request state in process memory**. State that must survive across separate requests, goroutines that originated elsewhere, or instance boundaries MUST be persisted to a shared backend (Firestore / GCS / Pub/Sub / Redis)
- **Allowed in-memory state**: only within a single continuous processing flow (e.g. variables within one HTTP request, one goroutine's local variables, one WebSocket connection's live buffer for the duration of that connection). As soon as the flow ends, the state must be gone or persisted
- **Forbidden patterns**:
  - In-memory registry/map keyed by ID that other requests lookup (e.g. `map[SessionID]*Handler` at package level)
  - Singleton caches of business data without a shared backend
  - Cross-goroutine coordination via channels at package scope
  - Assuming a WebSocket client is always on the same instance as the goroutine publishing to it
- **Required patterns**:
  - Firestore (or equivalent) as source of truth for all persistent state
  - Pub/Sub or Firestore snapshot listener for cross-instance event fan-out
  - Design reviews must explicitly verify multi-instance correctness for any new stateful component

### Test Requirements
- **EVERY code change MUST be accompanied by tests that verify the change**
- When adding new functionality, write tests that cover the new behavior
- When fixing a bug, write a test that reproduces the bug and verifies the fix
- When refactoring, ensure existing tests still pass and add tests if coverage gaps are found
- Do NOT consider a task complete until tests are written and passing

### Error Handling
- Use `github.com/m-mizutani/goerr/v2` for error handling
- Must wrap errors with `goerr.Wrap` to maintain error context
- Add helpful variables with `goerr.V` for debugging
- **NEVER check error messages using `strings.Contains(err.Error(), ...)`**
- **ALWAYS use `errors.Is(err, targetErr)` or `errors.As(err, &target)` for error type checking**
- Error discrimination must be done by error types, not by parsing error messages
- Tag errors with `goerr.T(errutil.TagXxx)` from `pkg/utils/errutil` where appropriate (see existing code for examples)
- **Use `errutil.Handle(ctx, err)` for error logging in background goroutines and fire-and-forget contexts** — it logs the error and sends it to Sentry in one call
  - BAD: `logger.Error("failed to do X", "error", err)`
  - GOOD: `errutil.Handle(ctx, goerr.Wrap(err, "failed to do X", goerr.V("id", id)))`

### Resource Cleanup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secmon-lab/warren](https://github.com/secmon-lab/warren) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
