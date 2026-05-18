---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Restriction & Rules

- In principle, do not trust developers who use this library from outside
  - Do not export unnecessary methods, structs, and variables
  - Assume that exposed items will be changed. Never expose fields that would be problematic if changed
  - Use `export_test.go` for items that need to be exposed for testing purposes
- When making changes, before finishing the task, always:
  - Run `go vet ./...`, `go fmt ./...` to format the code
  - Run `golangci-lint run ./...` to check lint error
  - Run `gosec -quiet ./...` to check security issue
  - Run tests to ensure no impact on other code
- All comment and character literal in source code must be in English
- Test files should have `package {name}_test`. Do not use same package name
- Test must be included in same name test file. (e.g. test for `abc.go` must be in `abc_test.go`)
- Use named empty structure (e.g. `type ctxHogeKey struct{}` ) as private context key
- Do not create binary. If you need to run, use `go run` command instead
- When a `tmp` directory is specified, search for files within the `./tmp` directory relative to the project root.
- **Naming Convention**: In Go, package names already express the subject, so avoid redundancy in method/struct names
  - Good: `openai.NewHistory()`, `claude.ToMessages()`, `gemini.ToContents()`
  - Bad: `openai.NewHistoryFromOpenAI()`, `claude.ToClaude()`, `gemini.ToGemini()`
- **Environment Variables**: Outside of test code, NEVER use `os.Getenv()` or `os.LookupEnv()` directly. Always use `github.com/urfave/cli/v3` for environment variable access
  - This ensures proper configuration management and testability
  - Test code may use `os.Getenv()` for test setup purposes


## Commands

### Development and Testing
- `task` or `task mock` - Generate mock files for testing (uses moq)
- `go test ./...` - Run all tests (MUST run before exiting tasks)
- `go test -v ./llm/openai/` - Run tests for specific package
- `go test -v ./path/to/package` - Run specific package tests when developing
- `go build ./...` - Build all packages
- `go mod tidy` - Clean up dependencies

### Test Execution
Tests may require API keys for integration testing:
- OpenAI: `OPENAI_API_KEY`
- Anthropic: `ANTHROPIC_API_KEY`
- Gemini: `GEMINI_PROJECT_ID`, `GEMINI_LOCATION`

When debugging tests, use the `trace/logger` package to enable LLM logging programmatically, or use `zenv` for environment-based configuration:
```bash
# Run tests with debug profile
zenv -p debug go test -v ./strategy/planexec/ -run TestName
```

### Code Quality
- MUST run `go test ./...` before completing any task
- Use `export_test.go` files to access internal packages for testing
- Clean up any test binaries after checking

## Architecture

**gollem** is a Go framework providing unified LLM access and agentic application building capabilities.

### Core Components

**Agent** (`gollem.go`) - Central orchestrator managing conversation loops, tool execution, and session management. Entry point: `gollem.New(llmClient, options...)`

**LLM Clients** (`llm/`) - Provider-specific implementations (OpenAI, Claude, Gemini) that all implement the `LLMClient` interface with `NewSession()` and `GenerateEmbedding()` methods.

**Session Management** (`session.go`) - Handles conversation state and message processing for each LLM interaction.

**Tool System** (`tool.go`) - Framework for LLM tool calling:
- `Tool` interface: individual tools with `Spec()` and `Run()` methods
- `ToolSet` interface: collections of tools (like MCP servers)
- JSON Schema-based parameter validation

**History Management** (`history.go`) - Cross-provider conversation history with versioning and portable serialization for stateless applications.

**MCP Integration** (`mcp/`) - Model Context Protocol support for connecting to external tool servers via stdio or Streamable HTTP.

**Facilitator** (`facilitator.go`) - Controls session termination and provides the default `respond_to_user` tool for conversation completion.

### Key Interfaces

```go
type LLMClient interface {
    NewSession(ctx context.Context, options ...SessionOption) (Session, error)
    GenerateEmbedding(ctx context.Context, dimension int, input []string) ([][]float64, error)
}

type Session interface {
    Generate(ctx context.Context, input []Input, opts ...GenerateOption) (*Response, error)
    Stream(ctx context.Context, input []Input, opts ...GenerateOption) (<-chan *Response, error)
    History() (*History, error)
    AppendHistory(*History) error
    CountToken(ctx context.Context, input ...Input) (int, error)
}

type Tool interface {
    Spec() ToolSpec
    Run(ctx context.Context, args map[string]any) (map[string]any, error)
}

type ToolSet interface {
    Specs(ctx context.Context) ([]ToolSpec, error)
    Run(ctx context.Context, name string, args map[string]any) (map[string]any, error)
}
```

### Structured Queries

- `Query[T]()` — One-shot structured query. Creates a new session, calls LLM with JSON schema, unmarshals into T.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m-mizutani/gollem](https://github.com/m-mizutani/gollem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
