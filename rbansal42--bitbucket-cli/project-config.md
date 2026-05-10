---
trigger: always_on
description: This document provides guidance for LLMs (Claude, GPT-4, etc.) and autonomous agents (Devin, OpenHands, Claude Code) to effectively use the `bb` CLI for Bitbucket Cloud operations.
---

# AGENTS.md - Guide for LLMs and Autonomous Agents

This document provides guidance for LLMs (Claude, GPT-4, etc.) and autonomous agents (Devin, OpenHands, Claude Code) to effectively use the `bb` CLI for Bitbucket Cloud operations.

## Core Concepts

### What is `bb`?

`bb` is an unofficial command-line interface for Bitbucket Cloud, similar to GitHub's `gh` CLI. It allows you to interact with Bitbucket repositories, pull requests, issues, pipelines, and more from the terminal.

### Resource Hierarchy

Bitbucket resources follow this hierarchy:

```
Workspace (e.g., "hudle")
├── Project (e.g., "PROJ")
│   └── Repository (e.g., "backend-api")
│       ├── Pull Requests
│       ├── Issues
│       ├── Pipelines
│       └── Branches
└── Snippets (workspace-level code snippets)
```

Understanding this hierarchy is essential:
- **Workspace**: Top-level container (like a GitHub organization)
- **Project**: Optional grouping of repositories within a workspace
- **Repository**: A git repository, always belongs to a workspace

### Repository Identification

Repositories are identified in two ways:

1. **Explicit**: `workspace/repo-name` format
   ```bash
   bb pr list --repo hudle/backend-api
   ```

2. **Implicit**: Detected from current git directory's remote
   ```bash
   cd ~/projects/backend-api
   bb pr list  # auto-detects from git remote
   ```

3. **Default workspace**: If a default workspace is configured, you can use just the repo name
   ```bash
   bb workspace set-default hudle
   bb pr list --repo backend-api  # resolves to hudle/backend-api
   ```

### Output Formats

Most commands support two output formats:

- **Human-readable** (default): Formatted tables for terminal display
- **JSON** (`--json` flag): Structured output for programmatic parsing

For programmatic use, always prefer `--json` output:
```bash
bb pr list --json
bb repo list --workspace myworkspace --json
```

---

## Authentication

### Checking Authentication Status

Before performing any operations, verify authentication:

```bash
bb auth status
```

Expected output when authenticated:
```
bitbucket.org
✓ Logged in to bitbucket.org account username (keyring)
  - Active account: true
  - Git operations protocol: ssh
  - Token: ATAT****...****
```

If not authenticated, the output will indicate no active account.

### Authentication Methods

#### Method 1: API Token (Recommended for CI/CD)

Atlassian API tokens are the simplest method, especially for automation:

```bash
bb auth login
# Select "API Token" when prompted
# Enter your Atlassian account email
# Paste your API token from https://id.atlassian.com/manage-profile/security/api-tokens
```

#### Method 2: OAuth (Interactive)

OAuth is browser-based and supports automatic token refresh:

```bash
bb auth login
# Select "OAuth" when prompted
# Complete browser-based authorization
```

### Environment Variables for CI/CD

For automated environments, set credentials via environment variables:

| Variable | Description |
|----------|-------------|
| `BB_TOKEN` | API token or OAuth access token |
| `BB_USERNAME` | Atlassian account email (for API token auth) |

Example CI/CD setup:
```bash
export BB_USERNAME="ci-bot@company.com"
export BB_TOKEN="ATATT3xFfGF0..."
bb pr list --repo myworkspace/myrepo
```

### Authentication Precedence

The CLI checks for credentials in this order:
1. Environment variables (`BB_TOKEN`, `BB_USERNAME`)
2. System keyring (stored by `bb auth login`)
3. Config file credentials

### Re-authentication

If authentication expires or becomes invalid:

```bash
bb auth logout
bb auth login
```

---

## Command Reference by Category

### Workspace Commands

```bash
# List workspaces you have access to
bb workspace list

# View workspace details
bb workspace view <workspace>

# List workspace members
bb workspace members <workspace>

# Set default workspace (eliminates need for --workspace flag)
bb workspace set-default <workspace>

# Show current default workspace
bb workspace set-default
```

### Repository Commands

```bash
# List repositories in a workspace
bb repo list --workspace <workspace>
bb repo list  # uses default workspace

# View repository details
bb repo view <workspace/repo>
bb repo view  # uses current git directory

# Clone a repository
bb repo clone <workspace/repo> [directory]

# Create a new repository
bb repo create --name <name> --workspace <workspace>
bb repo create --name <name>  # uses default workspace

# Delete a repository (requires confirmation)
bb repo delete <workspace/repo>

# Fork a repository
bb repo fork <workspace/repo> --workspace <destination-workspace>

# Sync fork with upstream
bb repo sync
```

### Pull Request Commands

```bash
# List pull requests
bb pr list --repo <workspace/repo>
bb pr list --state open|merged|declined|superseded
bb pr list  # uses current git directory

# View pull request details
bb pr view <number> --repo <workspace/repo>
bb pr view <number>  # uses current git directory

# Create a pull request
bb pr create --title "Title" --source <branch> --destination <branch>
bb pr create  # interactive mode

# Edit a pull request
bb pr edit <number> --title "New Title"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rbansal42/bitbucket-cli](https://github.com/rbansal42/bitbucket-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
