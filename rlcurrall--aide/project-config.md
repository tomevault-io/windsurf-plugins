---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **aide CLI** - a unified command-line tool for AI coding agents to interact with Jira, Azure DevOps, and GitHub APIs. Built with Bun/TypeScript, it provides a single binary that can be compiled for Windows, Linux, and macOS.

The CLI follows a hierarchical command structure: `aide <service> <action> [options]`

**Services:**

- `jira` - Jira ticket management (search, ticket, comment, comments, desc)
- `pr` - Pull request management (list, view, diff, create, update, comments, comment, reply)
- `plugin` - Claude Code plugin management (install, status, uninstall)

**Top-level Commands:**

- `prime` - Output aide context for session start hook
- `upgrade` - Upgrade aide to the latest version

## Development Commands

### Setup

```bash
bun install
```

### Running the CLI (Development)

```bash
# Using the dev script
bun run dev --help
bun run dev jira search "assignee = currentUser()"
bun run dev jira view PROJ-123
bun run dev pr list --status active
bun run dev pr comments --pr 24094 --latest 5

# Direct execution
bun run src/cli/index.ts --help
```

### Plugin Installation

```bash
# Install plugin (requires Claude CLI in PATH)
aide plugin install              # Install for current user (default)
aide plugin install --project    # Install to project scope
aide plugin install --local      # Install to local scope

# Check status and manage
aide plugin status
aide plugin uninstall --user
```

Or install manually from within Claude Code:

```bash
/plugin marketplace add rlcurrall/aide
/plugin install aide@aide-marketplace
```

### Building Binaries

```bash
bun run build           # Current platform
bun run build:win       # Windows (aide.exe)
bun run build:linux     # Linux (aide-linux)
bun run build:mac       # macOS ARM (aide-mac)
bun run build:all       # All platforms
```

Binaries are output to the `dist/` directory.

### Linting, Formatting, and Type Checking

```bash
bun run lint              # Run ESLint
bun run lint:fix          # Auto-fix linting issues
bun run format            # Format with Prettier
bun run format:check      # Check formatting
bunx tsc --noEmit         # Type check without emitting
```

## Architecture

### Directory Structure

```
src/
  cli/                    # CLI implementation
    index.ts              # Main entry point, service routing
    parser.ts             # Command line parsing (parseArgs, CommandContext type)
    help.ts               # Help text generation utilities
    commands/
      types.ts            # Command and ServiceRouter interfaces
      jira/               # Jira service commands
      pr/                 # Pull request commands (Azure DevOps/GitHub)
      plugin/             # Plugin management commands
      prime.ts            # Prime command
      upgrade.ts          # Upgrade command
  lib/                    # Shared libraries
    config.ts             # Configuration loading from env vars
    jira-client.ts        # Jira REST API client
    azure-devops-client.ts # Azure DevOps REST API client
    adf-to-md.ts          # Atlassian Document Format to Markdown
    md-to-adf.ts          # Markdown to Atlassian Document Format
    ado-utils.ts          # Git remote URL parsing
    cli-utils.ts          # CLI formatting helpers
    comment-utils.ts      # Comment filtering utilities
    types.ts              # TypeScript interfaces

commands/                 # Claude Code slash commands (for plugin)
  ticket.md               # /aide:ticket - Load Jira ticket context
  ticket-search.md        # /aide:ticket-search - Search Jira tickets
  ticket-comment.md       # /aide:ticket-comment - Add comment to ticket
  ticket-update.md        # /aide:ticket-update - Update ticket description
  pr-view.md              # /aide:pr-view - View PR details
  pr-diff.md              # /aide:pr-diff - View PR diff and changed files
  pr-comments.md          # /aide:pr-comments - Get PR comments
  pr-comment.md           # /aide:pr-comment - Post comment on PR
  pr-create.md            # /aide:pr-create - Create a PR
  pr-update.md            # /aide:pr-update - Update a PR
  pr-reply.md             # /aide:pr-reply - Reply to PR thread

skills/aide/SKILL.md      # Claude Code skill definition

.claude-plugin/           # Claude Code plugin metadata
  plugin.json             # Plugin manifest
  marketplace.json        # Marketplace listing info
```

### Command Architecture

Each service has a **router** that implements the `ServiceRouter` interface:

- Routes actions to appropriate command handlers
- Displays service-level help
- Handles unknown commands gracefully

Each command implements the `Command` interface:

- `name` - Command name
- `description` - Short description for help text
- `execute(ctx: CommandContext)` - Main execution function
- Returns exit code (0 = success, 1 = error)

### Key Design Patterns

**Service Routing:**
The main entry point (`cli/index.ts`) parses the command line and routes to service routers, which then route to individual commands.

**Auto-Discovery:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rlcurrall) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
