---
trigger: always_on
description: This file provides context and rules for AI-assisted coding platforms (GitHub Copilot, Cursor, Windsurf, Cline, etc.) working in this repository.
---

# AGENTS.md — AI Coding Assistant Instructions

This file provides context and rules for AI-assisted coding platforms (GitHub Copilot, Cursor, Windsurf, Cline, etc.) working in this repository.

## Project Overview

**ADO Pull Request Agent** is a .NET 10 console application that performs automated, AI-powered code reviews on Azure DevOps pull requests. It uses the GitHub Copilot SDK to orchestrate an AI model and connects to Azure DevOps via an MCP (Model Context Protocol) server to fetch PR diffs, then produces a structured Markdown review report.

## Tech Stack

- **Language:** C# (.NET 10)
- **Build system:** MSBuild / `dotnet` CLI
- **Solution file:** `ADOPullRequestAgent.slnx` (XML-based solution format)
- **Key dependencies:**
  - `GitHub.Copilot.SDK` — orchestrates Copilot CLI sessions and MCP server interactions
  - `System.CommandLine` — CLI argument parsing
  - `System.IO.Abstractions` — filesystem abstraction for testability
  - `Microsoft.Extensions.Logging.Console` — structured console logging
- **Container:** Multi-stage Dockerfile using `mcr.microsoft.com/dotnet/sdk:10.0` (build) and `mcr.microsoft.com/dotnet/runtime:10.0` (runtime)
- **CI/CD:** Azure DevOps pipeline (`.azdo/azure-pipeline.yaml`), GitHub Actions for Docker image builds (`.github/workflows/docker-buildandpush.yaml`)

## Repository Structure

```
├── AGENTS.md                        # This file
├── ADOPullRequestAgent.slnx         # Solution file
├── Dockerfile                       # Multi-stage Docker build for the agent
├── README.md                        # Project documentation
├── .azdo/
│   └── azure-pipeline.yaml          # Azure DevOps pipeline definition
├── .github/
│   └── workflows/
│       └── docker-buildandpush.yaml # GitHub Actions workflow
└── src/
    ├── ADOPullRequestAgent/         # Main console application
    │   ├── Program.cs               # Entry point — CLI parsing & orchestration
    │   ├── PullRequestAgent.cs      # Core agent logic — Copilot SDK session management
    │   ├── AgentOptions.cs          # Configuration POCO (model, CLI port)
    │   ├── pullreview.prompt        # System prompt for the AI code reviewer
    │   └── ADOPullRequestAgent.csproj
    └── GitHubCopilotCLIDocker/      # Docker image for GitHub Copilot CLI sidecar
        ├── Dockerfile
        └── README.md
```

## Coding Conventions

- **Nullable reference types** are enabled (`<Nullable>enable</Nullable>`). All reference types must be annotated; do not suppress warnings without justification.
- **Implicit usings** are enabled. Do not add redundant `using` statements for `System`, `System.Collections.Generic`, `System.Linq`, `System.Threading.Tasks`, etc.
- **File-scoped namespaces** are preferred for new files (e.g., `namespace ADOPullRequestAgent;`). Existing files may use block-scoped namespaces — follow the style of the file being edited.
- Use **XML documentation comments** (`<summary>`, `<param>`, `<returns>`, `<remarks>`) on all public and internal members.
- Follow standard C# naming conventions: `PascalCase` for types, methods, and properties; `camelCase` for local variables and parameters; `_camelCase` for private fields.
- Constructor injection for dependencies; use `IFileSystem` (from `System.IO.Abstractions`) instead of direct `System.IO` calls for testability.
- Prefer `async`/`await` throughout. Do not use `.Result` or `.Wait()` on tasks.
- Use `ArgumentNullException` for null-check guard clauses on constructor parameters.

## Build & Run

```bash
# Restore and build
dotnet build src/ADOPullRequestAgent/ADOPullRequestAgent.csproj

# Run (requires Copilot CLI running on port 4321 and env var COPILOT_GITHUB_TOKEN set)
dotnet run --project src/ADOPullRequestAgent -- \
  --ado-token "<token>" \
  --pull-request-id <id> \
  --organization-name <org> \
  --project-name <project> \
  --repository-name <repo> \
  --model "claude-sonnet-4.5"

# Docker build
docker build -t ado-pr-agent .
```

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `COPILOT_GITHUB_TOKEN` | Yes | GitHub PAT with Copilot scope — used by the Copilot SDK client |
| `ADO_MCP_AUTH_TOKEN` | Runtime | Azure DevOps access token — set internally by `PullRequestAgent` for the MCP server; passed via `--ado-token` CLI arg |

## Key Architecture Decisions

- **Copilot SDK orchestration:** `PullRequestAgent.RunAsync()` creates a `CopilotClient` → `Session` with two MCP servers (Azure DevOps for PR data, Microsoft Learn for best practices). The session streams events (`AssistantMessageEvent`, `SessionIdleEvent`, `SessionErrorEvent`) to build the review output.
- **System prompt:** The review persona and checklist live in `pullreview.prompt`, which is loaded at runtime. Edits to the review behavior should target this file, not C# code.
- **MCP server configuration:** The Azure DevOps MCP server is launched as a local child process via `npx @azure-devops/mcp@latest`. The ADO authentication token is passed through the `Env` dictionary on `McpLocalServerConfig`, eliminating the need for platform-specific shell wrappers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dstamand-msft/AzureDevOpsPullRequestAgent](https://github.com/dstamand-msft/AzureDevOpsPullRequestAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
