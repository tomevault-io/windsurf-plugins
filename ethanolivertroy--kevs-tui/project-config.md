---
trigger: always_on
description: This guide helps AI agents work effectively in this repository.
---

# AGENTS.md

This guide helps AI agents work effectively in this repository.

## Project Overview

**kevs-tui** is a Terminal UI (TUI) application for browsing CISA Known Exploited Vulnerabilities (KEV) catalog with an integrated AI assistant (KEVin).

- **Language**: Go 1.26.0
- **Module**: `github.com/ethanolivertroy/kevs-tui`
- **Main Framework**: Bubble Tea (TUI), Google ADK (Agent)
- **Purpose**: Interactive security vulnerability browser with AI-powered analysis

## Essential Commands

### Building
```bash
go build ./...                    # Build all packages
go build -o kev .                 # Build main binary
```

### Testing
```bash
go test ./...                     # Run all tests
go test -coverprofile=coverage.out -covermode=atomic ./...  # With coverage
```

### Linting
```bash
golangci-lint run                 # Run linters (v2.9.0)
```

### Running
```bash
go run .                          # Run TUI (KEV browser + agent sidebar)
go run . agent                    # Interactive agent chat only
go run . agent "Microsoft vulns"   # One-shot query
go run . serve                    # Start A2A server on port 8001
go run . serve --port 9000        # A2A server on custom port
```

### Environment Variables
```bash
# LLM Provider Selection
LLM_PROVIDER=gemini               # Options: gemini, vertex, ollama, openrouter (default: gemini)
LLM_MODEL=gemini-2.0-flash        # Model name

# Gemini
GEMINI_API_KEY=your-key

# Vertex AI
VERTEX_PROJECT=gcp-project-id
VERTEX_LOCATION=us-central1

# Ollama
OLLAMA_URL=http://localhost:11434

# OpenRouter
OPENROUTER_API_KEY=your-key
```

## Code Organization

```
.
├── main.go                      # Entry point, AppModel, TUI layout
├── cmd/                         # Subcommand handlers
│   ├── agent.go                 # Agent CLI mode
│   └── serve.go                 # A2A server mode
└── internal/                    # Application logic
    ├── tui/                     # Bubble Tea models (app.go, styles.go, keys.go)
    ├── model/                   # Data structures (Vulnerability, EPSS, CVSS)
    ├── api/                     # External API clients (KEV, EPSS, NVD)
    ├── agent/                   # KEVin AI agent (Google ADK integration)
    ├── llm/                     # LLM provider abstraction
    ├── chat/                    # Chat interface for agent
    ├── server/                  # A2A server implementation
    ├── grc/                     # GRC compliance tools (NIST 800-53, FedRAMP)
    ├── palette/                 # Command palette (Ctrl+K)
    └── export/                  # Export to JSON/CSV/Markdown
```

## Code Patterns

### Bubble Tea Architecture

Models must implement the three core methods:

```go
type Model struct {
    // State fields
}

func (m Model) Init() tea.Cmd {
    // Initialize and return initial commands
    return tea.Batch(cmd1, cmd2)
}

func (m Model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    // Handle messages and update state
    switch msg := msg.(type) {
    case tea.KeyMsg:
        // Handle key presses
    case tea.WindowSizeMsg:
        // Handle resize
    case CustomMsg:
        // Handle custom messages
    }
    return m, nil
}

func (m Model) View() string {
    // Return rendered string
}
```

### Message Passing

Custom messages for inter-component communication:

```go
type VulnsLoadedMsg struct {
    Vulns []model.Vulnerability
}

type ErrorMsg struct {
    Err error
}

// In Update():
case VulnsLoadedMsg:
    m.vulns = msg.Vulns
    return m, nil
```

### Error Handling Pattern

Always wrap errors with context:

```go
if err != nil {
    return nil, fmt.Errorf("failed to fetch data: %w", err)
}
```

### LLM Provider Abstraction

The `internal/llm` package provides a unified interface for multiple LLM providers:

```go
cfg := llm.ConfigFromEnv()
if err := cfg.Validate(); err != nil {
    return err
}
model, err := llm.NewModel(ctx, cfg)
```

Supported providers:
- `gemini` (default): Requires `GEMINI_API_KEY`
- `vertex`: Requires `VERTEX_PROJECT` and `VERTEX_LOCATION`
- `ollama`: Requires `OLLAMA_URL` (defaults to localhost:11434)
- `openrouter`: Requires `OPENROUTER_API_KEY`

## Naming Conventions

### Files and Packages
- Package names: lowercase, single word (`tui`, `model`, `api`)
- File names: lowercase, underscore_separated (`app_test.go`, `vulnerability.go`)
- Test files: `{name}_test.go` in same package as code

### Types and Functions
- Types: PascalCase (`Model`, `Vulnerability`, `AppState`)
- Functions: PascalCase if exported (`NewModel()`, `FetchVulns()`)
- Functions: camelCase if private (`applySortAndFilter()`, `calculateStats()`)

### Constants
- Constants: PascalCase or UPPER_SNAKE_CASE for constants
  - PascalCase: `SortByDateAdded`, `ViewList`
  - UPPER_SNAKE_CASE: `AgentPanelWidth`, `HeaderHeight`

### Messages
- Messages: PascalCase ending with `Msg` (`VulnsLoadedMsg`, `ErrorMsg`)

## Testing Approach

### Table-Driven Tests

Use table-driven tests for multiple test cases:

```go
func TestNVDURL(t *testing.T) {
    tests := []struct {
        name     string
        cveID    string
        expected string
    }{
        {"standard CVE", "CVE-2024-1234", "https://nvd.nist.gov/vuln/detail/CVE-2024-1234"},

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethanolivertroy/kevs-tui](https://github.com/ethanolivertroy/kevs-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
