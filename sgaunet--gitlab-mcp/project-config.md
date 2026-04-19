---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) server that provides GitLab integration tools for Claude Code. It's implemented as a Go application that communicates via JSON-RPC 2.0 over stdin/stdout, providing tools for GitLab project management:

- `list_issues`: Lists issues for a GitLab project using project path with filtering options
- `create_issues`: Creates new issues with title, description, labels (validated), and assignees
- `update_issues`: Updates existing issues (title, description, state, labels (validated), assignees)
- `list_labels`: Lists project labels with optional filtering and counts
- `add_issue_note`: Adds notes/comments to existing issues
- `get_project_description`: Gets the current description of a GitLab project
- `update_project_description`: Updates the description of a GitLab project
- `get_project_topics`: Gets the current topics/tags of a GitLab project
- `update_project_topics`: Updates the topics/tags of a GitLab project (replaces all existing topics)
- `list_epics`: Lists epics for a GitLab group (Premium/Ultimate tier)
- `create_epic`: Creates new epics in a GitLab group with title, description, labels (validated), dates, and confidentiality (Premium/Ultimate tier)
- `get_latest_pipeline`: Gets the latest pipeline for a GitLab project with optional ref (branch/tag) filtering
- `list_pipeline_jobs`: Lists all jobs for a GitLab pipeline with filtering options (status, stage)
- `get_job_log`: Retrieves complete log output for a specific CI/CD job with job metadata
- `download_job_trace`: Downloads CI/CD job logs to local files for offline analysis and archiving
- `list_merge_requests`: Lists merge requests for a GitLab project with filtering options (state, labels, author, search)
- `create_merge_request`: Creates a new merge request with title, source/target branches, description, labels, assignees, and reviewers
- `get_merge_request`: Gets details of a specific merge request by IID
- `update_merge_request`: Updates an existing merge request (title, description, state, labels, assignees, reviewers)
- `merge_merge_request`: Merges an approved merge request with squash and remove-source-branch options
- `get_merge_request_diff`: Gets the unified diff for a merge request showing actual code changes per file
- `approve_merge_request`: Approves a merge request with optional SHA verification
- `add_merge_request_note`: Adds a comment/note to a merge request

## Architecture

The codebase follows a clean architecture pattern with clear separation of concerns:

### Main Components
- **main.go**: MCP server initialization, protocol setup, and tool registration
  - Each tool has a dedicated `setup*Tool()` function that creates the tool definition
  - Request handlers are implemented as `handle*Request()` functions with input validation
  - Parameter extraction is separated into `extract*Options()` helper functions
  - All tools follow the same pattern: validate → extract → call app → marshal → return

- **internal/app/app.go**: Business logic for GitLab API integration
  - `App` struct holds GitLab client, configuration, and logger
  - Public methods for each tool operation (e.g., `ListProjectIssues`, `CreateProjectIssue`)
  - Private helper methods for validation
  - Interface-based design for testability via dependency injection

- **internal/app/merge_requests.go**: Business logic for merge request operations
  - CRUD operations, merging, diffs, approvals, and notes for merge requests
  - Converts GitLab API types to internal MergeRequest representation

- **merge_request_tools.go**: MCP tool definitions and handlers for merge request operations
  - Tool registration functions (`setup*Tool()`) for all 8 merge request tools
  - Parameter parsing helpers (`parseListMROptions`, `parseUpdateMROptions`, `parseLabels`, `parseIDArray`)

- **internal/app/interfaces.go**: Interface definitions for GitLab client abstraction
  - Defines service interfaces matching GitLab API structure
  - Enables mocking in tests without external dependencies

- **internal/app/client.go**: Production wrapper implementing GitLabClient interface
  - Wraps the official `gitlab.Client` to match interface contracts
  - Provides access to all GitLab service implementations

- **internal/logger**: Structured logging utilities with configurable levels
  - Uses Go's `log/slog` for structured logging
  - Outputs to stderr to avoid interfering with MCP stdin/stdout communication

### Key Architectural Patterns
- **Dependency Injection**: `NewWithClient()` allows injecting mocked clients for testing
- **Interface Segregation**: Each GitLab service (Issues, Projects, Labels, etc.) has its own interface
- **Clean Error Handling**: Static error variables with wrapped context in error messages
- **MCP Protocol**: Uses `github.com/mark3labs/mcp-go` library for JSON-RPC 2.0 over stdio

The server uses the official GitLab Go client library (`gitlab.com/gitlab-org/api/client-go`) and supports both GitLab.com and self-hosted instances.

### Usage Examples

**Listing issues:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sgaunet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
