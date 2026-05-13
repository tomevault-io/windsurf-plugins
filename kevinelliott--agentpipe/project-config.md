---
trigger: always_on
description: AgentPipe is a CLI and TUI application that orchestrates conversations between multiple AI agent CLIs (Claude, Gemini, Qwen, Codex, Ollama). It allows different AI tools to communicate in a shared "room" with various conversation modes.
---

# AgentPipe Project Memory

## Project Overview
AgentPipe is a CLI and TUI application that orchestrates conversations between multiple AI agent CLIs (Claude, Gemini, Qwen, Codex, Ollama). It allows different AI tools to communicate in a shared "room" with various conversation modes.

## Key Technical Details

### Go Version
- **IMPORTANT**: Requires Go 1.24+ (go.mod specifies 1.24.0)
- GitHub Actions workflows must use Go 1.24
- All dependencies are compatible with Go 1.24

### Health Check Configuration
- Default timeout: 5 seconds (increased from 2 seconds)
- Claude CLI needs longer startup time
- Flag: `--health-check-timeout` to customize
- Flag: `--skip-health-check` to bypass

### Directory Structure
- Chat logs: `~/.agentpipe/chats/` (default)
- Homebrew formula: `Formula/` (NOT Formulae/)
- Config examples: `examples/`

### CI/CD Configuration

#### Linting
- Use golangci-lint-action@v6 with golangci-lint v1.x (latest stable)
- GitHub Action version parameter: `version: latest` (downloads latest v1.x)
- **IMPORTANT**: Config file (`.golangci.yml`) uses v1.x format (no version field)
- **Note**: Removed `nakedret` linter (removed in v2.x, compatible with v1.x)
- **Status**: v2.x requires config schema changes; staying on v1.x is more stable
- **Local linting**: Install latest golangci-lint v1.x
  - Install: `brew install golangci-lint`
  - Or: `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest`
- Configuration structure: `linters-settings:` for linter config, `issues.exclude-rules:` for exclusions
- Cognitive complexity threshold: 30
- Excluded from complexity checks: pkg/tui/, pkg/adapters/, pkg/orchestrator/

#### Testing
- Windows test fix: Single-line command (no multiline)
- Command: `go test -v -race ./...`
- No coverage profile to avoid Windows parsing issues

#### Releases
- Triggered on tags: `v[0-9]+.[0-9]+.[0-9]+`
- Requires `HOMEBREW_TAP_TOKEN` secret for formula updates
- Token needs `repo` scope for cross-repo access

### Common Issues & Fixes

1. **Claude CLI health check timeout**
   - Solution: Increased default timeout to 5 seconds
   - User can use `--health-check-timeout 10` for longer

2. **Windows test failures - multiline commands**
   - Issue: Multiline commands break on Windows
   - Solution: Use single-line test command

3. **Windows test failures - timer resolution**
   - Issue: `time.Since()` can return 0 for very fast operations due to Windows timer granularity (~15.6ms default)
   - Solution: Add delays (≥20ms) to mock agents in tests to ensure measurable durations
   - Example: `agent.sendDelay = 20 * time.Millisecond` in integration tests
   - Windows timer is much coarser than Unix (typically 1ms or better)

4. **Homebrew formula updates failing**
   - Issue: GitHub Actions bot lacks permissions
   - Solution: Add `HOMEBREW_TAP_TOKEN` secret with repo scope

5. **Linting errors**
   - Empty branches: Add comment or `_ = err`
   - Imports: Use `goimports -local github.com/kevinelliott/agentpipe`
   - Deprecated methods: Updated viewport scroll methods

### Agent Adapters
AgentPipe supports two types of agent adapters:

**CLI-Based Adapters** (Claude, Gemini, Qwen, Continue, etc.):
- Execute external CLI tools via `exec.Command`
- Each adapter must implement full `Agent` interface
- Methods: `Initialize(config)`, `IsAvailable()`, `HealthCheck(ctx)`, `SendMessage(ctx, messages)`, `StreamMessage(ctx, messages, writer)`, `GetCLIVersion()`
- **Continue CLI** (`cn`): Uses `-p` flag for prompt, `--model` for model selection, `--silent` to strip think tags

**API-Based Adapters** (OpenRouter, v0.6.0+):
- Direct HTTP API integration without CLI dependencies
- Same `Agent` interface but uses HTTP client instead of exec
- `IsAvailable()` checks for API key instead of CLI binary
- `GetCLIVersion()` returns "N/A (API)" for API-based agents
- Benefits: No CLI installation, lower latency, real token counts from API

### TUI Features
- Three panels: agents list, conversation, user input
- Color-coded agent messages with badges
- Real-time metrics display (duration, tokens, cost)
- Modal system for agent details
- User participation with 'u' key

### Configuration
YAML config supports:
- Multiple agents with custom prompts
- Orchestrator modes: round-robin, reactive, free-form
- Logging configuration
- Turn limits and timeouts
- **Streaming bridge configuration** (v0.3.0+):
  - Bridge enabled status, URL, API key, timeout, retry attempts, log level
  - Defaults: disabled, 10s timeout, 3 retries
  - Environment variables override config file: `AGENTPIPE_STREAM_ENABLED`, `AGENTPIPE_STREAM_URL`, `AGENTPIPE_STREAM_API_KEY`

### Streaming Bridge (v0.3.0+)
**Overview:**
- Opt-in real-time conversation streaming to AgentPipe Web (https://agentpipe.ai)
- Four event types: `conversation.started`, `message.created`, `conversation.completed`, `conversation.error`
- Non-blocking async HTTP implementation using goroutines - never blocks conversations
- Privacy-first: disabled by default, API keys never logged, clear disclosure

**Architecture:**
- Package: `internal/bridge/`
- Components:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinelliott/agentpipe](https://github.com/kevinelliott/agentpipe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
