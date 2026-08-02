---
trigger: always_on
description: This is a **Slack Bot** written in **Go** that improves development team workflows with integrations for Jenkins, GitHub, GitLab, and Jira. The bot supports custom commands, macros, cron jobs, and flexible project-specific functionality.
---

# Instructions for Slack Bot Repository

## Repository Overview

This is a **Slack Bot** written in **Go** that improves development team workflows with integrations for Jenkins, GitHub, GitLab, and Jira. The bot supports custom commands, macros, cron jobs, and flexible project-specific functionality.

- **Language**: Go (requires Go 1.25+)
- **Type**: Slack application with Socket Mode support
- **Size**: ~50+ Go packages across bot/, command/, client/, and cmd/ directories
- **Architecture**: Modular command-based bot with plugin system
- **Runtime**: Standalone Go binary or Docker container

## Build and Validation Instructions

### Prerequisites
- **Go 1.25 or later**
- **Make** (for build targets)
- **Docker** (optional, for containerized builds)
- **golangci-lint** (for linting)

### Critical Build Steps

1. **Always sync vendor directory first**:
   ```bash
   make dep
   ```
   **Note**: The project uses vendored dependencies. Vendor inconsistencies will cause build failures.

2. **Build the main application**:
   ```bash
   make build/slack-bot
   ```
   Or build everything:
   ```bash
   make all
   ```

3. **Alternative Docker build**:
   ```bash
   make docker-build
   ```

### Testing Commands
- **Run all tests**: `make test`
- **Race detection**: `make test-race`
- **Coverage**: `make test-coverage` (creates `./build/cover.html`)
- **Benchmarks**: `make test-bench`

### Development Commands
- **Run bot locally**: `make run`
- **Run CLI tool**: `make run-cli` (requires `config.yaml`)
- **Live reload**: `make run-live-reload` (uses air for hot reloading)
- **Generate mocks**: `make mocks`
- **Lint code**: `make lint` (auto-fixes issues)

### Common Build Issues
- **Vendor inconsistency**: Always run `make dep` after dependency changes
- **Go version mismatch**: Ensure go toolchain matches go version
- **Missing config**: CLI commands require `config.yaml` file
- **Build flags**: Uses trimpath and ldflags with git version injection

## Project Architecture and Layout

### Main Entry Point
- **`cmd/bot/main.go`**: Primary application entry point, calls `bot/app.Run()`
- **`cmd/cli/main.go`**: CLI tool for bot administration

### Core Directories
- **`bot/`**: Core bot functionality, configuration, listeners, message handling
- **`command/`**: All bot commands organized by feature (jenkins/, jira/, games/, etc.)
- **`client/`**: External service integrations (Slack, Bitbucket, VCS clients)
- **`cmd/`**: Command-line applications (bot and cli tools)

### Command and Matcher Structure

Commands follow a consistent pattern with three main components:

#### 1. Command Interface Implementation
Each command implements the `bot.Command` interface with these methods:
- `GetMatcher() matcher.Matcher` - Defines what triggers the command
- `GetHelp() []bot.Help` - Provides help documentation
- `IsEnabled() bool` - Optional feature flag support

#### 2. Matcher Types
Commands use different matcher types from `bot/matcher/`, like:
- **`TextMatcher`**: Exact text matches (e.g., "list jenkins nodes")
- **`RegexpMatcher`**: Regex patterns with named groups (e.g., `(?P<resource>...)`)
- **`GroupMatcher`**: Combines multiple matchers for complex commands

Example from Jenkins nodes command:
```go
func (c *nodesCommand) GetMatcher() matcher.Matcher {
    return matcher.NewTextMatcher("list jenkins nodes", c.listNodes)
}
```

Example from Pool commands with regex and groups:
```go
return matcher.NewGroupMatcher(
    matcher.NewRegexpMatcher("pool lock\\b( )?(?P<resource>...)(?P<reason>...)", c.lockResource),
    matcher.NewRegexpMatcher("pool unlock( )?(?P<resource>...)", c.unlockResource),
    matcher.NewTextMatcher("pool locks", c.listUserResources),
)
```

#### 3. Handler Functions
Handler functions receive:
- `match matcher.Result` - Contains captured groups from regex
- `message msg.Message` - The incoming Slack message

Access captured groups with `match.GetString("groupName")`:
```go
func (c *poolCommands) lockResource(match matcher.Result, message msg.Message) {
    resourceName := match.GetString("resource")
    reason := match.GetString("reason")
    // ... command logic
}
```

#### 4. Async Commands
Commands can implement `RunAsync(ctx *util.ServerContext)` for background tasks:
```go
func (c *poolCommands) RunAsync(ctx *util.ServerContext) {
    ctx.RegisterChild()
    defer ctx.ChildDone()
    // ... background logic with proper shutdown support
}
```

### Configuration
- **`config.yaml`**: Runtime configuration (use `config.example.yaml` as template)
- **Required**: Slack token (`slack.token`) and socket token (`slack.socket_token`)
- **Structure**: YAML-based with sections for slack, jenkins, jira, pool, etc.

### CI/CD Pipeline (`.github/workflows/test.yaml`)
1. **Multi-platform testing** (Ubuntu, macOS, Windows)
2. **Multi-version Go testing** (1.24.x, 1.25.x)
3. **Build validation** using `make build/slack-bot`
4. **Race testing** using `make test-race`
5. **Coverage** using `make test-coverage`
6. **Examples build** in `examples/custom_commands`

### Validation Steps Before Check-in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [innogames/slack-bot](https://github.com/innogames/slack-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
