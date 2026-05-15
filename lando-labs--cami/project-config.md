---
trigger: always_on
description: **CAMI Source Code Repository**
---

# CAMI Development - Claude Agent Management Interface

**CAMI Source Code Repository**

This is the CAMI development repository. You are working on the Go codebase that powers CAMI's MCP server and CLI.

---

## Claude Context: CAMI Developer Assistant

**You are assisting with CAMI development** - working on the Go codebase that implements the Model Context Protocol (MCP) server and CLI for Claude Code agent management.

**Your Focus Areas:**

1. **Go Development** - Write clean, idiomatic Go code following the project's patterns and conventions.

2. **MCP Implementation** - Understand and work with the MCP protocol implementation in `internal/mcp/`.

3. **Agent Management Logic** - Work on agent loading, deployment, source management in `internal/agent/`, `internal/deploy/`, etc.

4. **Cross-Platform Support** - Ensure code works on macOS, Linux, and Windows.

5. **Testing & Quality** - Write tests, use the QA agent for coverage, maintain code quality.

**Development Workflow:**

- Use `go run ./cmd/cami` for local testing
- The `.mcp.json` in this repo uses `go run` for zero-setup development
- Run `make build` to compile binary
- Run `make test` for tests
- Run `make lint` for code quality checks
- Use deployed agents (qa, agent-architect) for development tasks

**Remember:** This is the CAMI source code repo. Users install CAMI via releases and get a clean `~/cami-workspace/` workspace with templates from `install/templates/`.

---

## Architecture Overview

### Single Binary, Dual Modes

```bash
# MCP Server Mode (primary) - for Claude Code
$ cami --mcp
# Runs as MCP server on stdio for Claude Code integration

# CLI Mode (secondary) - for scripting and quick checks
$ cami list
$ cami deploy frontend backend ~/projects/my-app
$ cami scan ~/projects/my-app
```

### User Workspace Structure

CAMI creates a user workspace at `~/cami-workspace/` during installation:

```
~/cami-workspace/                          # User workspace (created by installer)
├── CLAUDE.md                    # User-facing CAMI documentation
├── README.md                    # User quick start guide
├── .mcp.json                    # Local MCP config
├── .gitignore                   # Git ignore rules
├── config.yaml                  # CAMI configuration
├── .claude/
│   └── agents/                  # CAMI's own agents
├── sources/                     # Agent sources
│   ├── my-agents/              # User's custom agents
│   ├── team-agents/            # (if added)
│   └── fullstack-guild/        # Example: guild added via add_source

/usr/local/bin/cami             # Binary (on PATH)
```

**Key Concepts:**
- User workspace is separate from source code repo
- Templates live in `install/templates/` in this repo
- Installer copies templates to `~/cami-workspace/` during setup
- Users can optionally track `~/cami-workspace/` with Git

### Configuration Format

User's `~/cami-workspace/config.yaml`:

```yaml
version: "1"
install_timestamp: 2025-01-18T10:30:00Z  # When CAMI was installed
setup_complete: true                      # Whether initial setup is complete
agent_sources:
  - name: team-agents
    type: local
    path: ~/cami-workspace/sources/team-agents
    priority: 50
    git:
      enabled: true
      remote: git@github.com:yourorg/team-agents.git

  - name: my-agents
    type: local
    path: ~/cami-workspace/sources/my-agents
    priority: 10
    git:
      enabled: false

deploy_locations:
  - name: my-project
    path: ~/projects/my-project

default_projects_dir: ~/projects  # Where new projects are created by default
```

**New in v0.3.2**:
- `install_timestamp`: Automatically set during installation (used for robust fresh install detection)
- `setup_complete`: False on fresh install, true after adding real agent sources
- Existing configs are automatically migrated on first load

**Priority-based deduplication**: When the same agent exists in multiple sources, the lowest priority number wins (my-agents: 10 > team-agents: 50). Priority 1 = highest, 100 = lowest.

## Development Setup

### This Repository (Dev Mode)

This repo has `.mcp.json` configured for development:

```json
{
  "mcpServers": {
    "cami": {
      "command": "go",
      "args": ["run", "./cmd/cami", "--mcp"]
    }
  }
}
```

Open this directory in Claude Code and CAMI runs automatically via `go run`.

### Building

```bash
# Build for current platform
make build

# Build for all platforms
make release-all

# Package releases with installer
make package

# Install locally (creates ~/cami-workspace/ workspace)
make install
```

### Testing

```bash
# Run tests
make test

# Run linters
make lint

# Test installation
make install
cd ~/cami-workspace
claude  # Test user experience
```

## Installation Templates

User workspace files are in `install/templates/`:

```
install/
├── templates/
│   ├── CLAUDE.md        # User-facing CAMI persona doc
│   ├── README.md        # User quick start guide
│   ├── .gitignore       # Git ignore for user workspace
│   └── .mcp.json        # Local MCP config (uses 'cami' on PATH)
└── install.sh           # Installation script
```

When modifying the user experience:
- Update templates in `install/templates/`
- Update installer in `install/install.sh`
- Test with `make install` and check `~/cami-workspace/`

## Project Structure

```
cami/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lando-labs/cami](https://github.com/lando-labs/cami) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
