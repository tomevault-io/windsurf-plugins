---
trigger: always_on
description: This file provides guidance to AI coding assistants (like Claude Code, GitHub Copilot, etc.) when working with code in this repository.
---

# AGENT.md

This file provides guidance to AI coding assistants (like Claude Code, GitHub Copilot, etc.) when working with code in this repository.

## Overview

**OpenAI Agents Go SDK** - Community-maintained Go SDK for building AI agents with OpenAI's API. Provides multi-agent workflows, tool calling, handoffs, and structured outputs with full type safety.

- **Module**: `github.com/MitulShah1/openai-agents-go`
- **Package**: `agents`
- **Go Version**: 1.24+

## Mandatory Verification

After any code modification, run the full verification stack before considering work complete:

```bash
make check    # Runs fmt, vet, lint, and tests
go test -v -race ./...
```

Rerun checks after fixing failures. All checks must pass before pull requests.

## Build & Development Commands

```bash
# Build and test
go build ./...                    # Build all packages
go test ./...                     # Run all tests
go test -v -race ./...            # Race condition detection
go test -cover ./...              # Coverage analysis

# Run examples (requires OPENAI_API_KEY)
export OPENAI_API_KEY="your-key"
go run examples/01_basic/main.go
go run examples/06_structured_output/main.go

# Code quality (run before commits)
go fmt ./...                      # Format code
go vet ./...                      # Static analysis
golangci-lint run                 # Comprehensive linting

# Makefile targets (recommended)
make check                        # Run all checks (fmt, vet, lint)
make test                         # Run tests with coverage
```

## Repository Structure

```
.
├── agent.go               # Agent type and configuration
├── runner.go              # Agent execution orchestration
├── tool.go                # Tool interface and implementations
├── config.go              # Run configuration options
├── types.go               # Shared types (Result, Step, Usage, etc.)
├── errors.go              # Structured error types
├── guardrail/             # Input/output validation framework
│   └── builtin/          # Built-in guardrails (PII, URL, regex)
├── session/              # Conversation persistence
│   ├── memory.go         # In-memory session storage
│   └── file.go           # File-based session storage
├── internal/
│   └── jsonschema/        # JSON Schema builder for structured outputs
├── examples/              # Usage examples (numbered by complexity)
│   ├── 01_basic/          # Hello world agent
│   ├── 02_tools/          # Tool calling
│   ├── 03_handoffs/       # Agent handoffs
│   ├── 04_lifecycle_hooks/# OnBeforeRun/OnAfterRun hooks
│   ├── 05_config_usage/   # Run configuration
│   ├── 06_structured_output/  # JSON schema outputs
│   ├── 07_complex_schema/ # Nested schemas
│   ├── 08_guardrails_demo/    # Guardrails demonstration
│   ├── 09_sessions_demo/      # Sessions demonstration
│   └── 10_advanced_v02/       # Production chatbot (v0.2.0)
├── .github/workflows/     # CI/CD pipelines
├── AGENT.md              # This file
├── README.md             # User-facing documentation
└── ROADMAP.md            # Future features

**Data Flow**:
1. User → `Runner.Run()` → OpenAI API (via `github.com/openai/openai-go`)
2. API Response → Tool Execution → Agent Handoffs → Final Result
3. Structured Outputs: Schema → Validation → Type-safe JSON

## Code Conventions

- **Idiomatic Go**: Use `gofmt` formatting, standard naming conventions
- **Interface-driven**: All tools implement `Tool` interface
- **Error handling**: Use `fmt.Errorf` with `%w` verb for wrapping, include contextual information
- **Context-first**: All blocking functions accept `context.Context` as first parameter
- **Cyclomatic complexity**: Keep functions under complexity 30 (gocyclo threshold); higher acceptable for table-driven tests and orchestration code
- **Naming patterns**: 
  - Exported types use descriptive names (`Agent`, `Runner`, `Tool`)
  - Options use functional options pattern
  - Errors use `ErrXxx` or `XxxError` naming
- **No unnecessary exports**: Keep internal packages unexported unless needed by external consumers

## Key Design Patterns

1. **Functional Options Pattern**: Used throughout for configuration
   ```go
   agent := NewAgent("name")
   agent.Instructions = "helpful assistant"
   ```

2. **Tool Interface**: Abstract function execution
   ```go
   type Tool interface {
       ToParam() openai.ChatCompletionToolParam
       Execute(args string, ctx ContextVariables) (any, error)
   }
   ```

3. **Handoff Pattern**: Special tool result type for agent transfers
   ```go
   type Handoff struct { Agent *Agent }
   ```

4. **Structured Outputs**: Fluent schema builder
   ```go
   schema := jsonschema.Object().
       WithProperty("field", jsonschema.String()).
       WithRequired("field")
   ```

## Testing Practices

### Unit Tests
- **Mandatory**: Add or update unit tests for any code change unless truly infeasible; if tests can't be added, explain why in PR
- **Table-driven tests**: Use for multiple test cases
- **Mock external calls**: Don't call OpenAI API in tests (use fixtures if needed)
- **Test naming**: `TestFunctionName_Scenario` format
- **Coverage target**: Aim for >80% on core logic

### Running Tests
```bash
# All tests
go test -v ./...


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MitulShah1/openai-agents-go](https://github.com/MitulShah1/openai-agents-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
