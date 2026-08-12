---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A .NET 10 dotnet tool that serves a real-time Kanban dashboard for monitoring Claude Code agent tasks, sessions, and subagents. It watches `~/.claude/` directories, broadcasting changes via Server-Sent Events to a browser-based board.

## Build & Test

```powershell
# Build (Release mode enforces all analyzer rules as errors)
dotnet build -c Release

# Run tests (115 tests across 11 test classes)
dotnet test

# Run the dashboard locally
dotnet run --project src/Atc.Claude.Kanban -- --open

# Custom port
dotnet run --project src/Atc.Claude.Kanban -- --port 8080

# Point at a synthetic fixture instead of ~/.claude (used to regenerate README
# screenshots without real session data — see scripts/seed-demo-fixture.ps1)
dotnet run --project src/Atc.Claude.Kanban -- --dir <path>
```

## Repository Structure

```
src/Atc.Claude.Kanban/
  Program.cs                    # Entry point, CLI args, auto-port discovery, WebApplication wiring
  CliOptions.cs                 # Parsed CLI argument record
  EndpointDefinitions/          # Atc.Rest.MinimalApi IEndpointDefinition implementations
    SessionEndpointDefinition   # /api/sessions, /api/sessions/{id}, /api/sessions/{id}/agents, /api/sessions/{id}/messages,
                                #   /api/sessions/{id}/messages/{uuid}/image/{blockIndex}, /api/sessions/{id}/tool-stats, /api/sessions/{id}/usage
    TaskEndpointDefinition      # /api/tasks/all, PUT/DELETE/POST task operations
    TeamEndpointDefinition      # /api/teams/{name}
    ProjectEndpointDefinition   # /api/projects
    PlanEndpointDefinition      # /api/plans/{slug}, /api/plans/{slug}/open
    SubagentEndpointDefinition  # /api/sessions/{id}/subagents, /api/sessions/{id}/subagents/{agentId}/messages
    SseEndpointDefinition       # /api/events (SSE), /api/version, /api/cache/clear
    UtilityEndpointDefinition   # /api/open-folder, /api/open-in-editor
  Contracts/
    Models/                     # ClaudeTask, SessionInfo, TeamConfig, SubagentInfo, MessageEntry, SessionTokenUsage,
                                #   AnswerPayload/AnswerQuestion/AnswerOption (AskUserQuestion), MessageImage, etc.
    Events/                     # SseNotification, FileChangeEvent
    Responses/                  # ErrorResult, UpdateResult, ToolStatsResponse/ToolStat, UsageResponse/UsageRow, etc.
    Parameters/                 # [AsParameters] records (SessionIdParameters, UserImageParameters, etc.)
  Helpers/
    PathHelper                  # Path traversal prevention (shared by TaskService, PlanService)
    TokenCostCalculator         # Model-aware token cost calculation (Opus/Sonnet/Haiku pricing)
  Extensions/                   # ServiceCollectionExtensions, WebApplicationExtensions
  Services/
    SessionService              # Session discovery from tasks/ + metadata from projects/, snapshots
    TaskService                 # Task CRUD, dependency validation, notes, snapshots
    TeamService                 # Team config reading with 5s cache TTL
    PlanService                 # Plan markdown reading
    SubagentService             # Subagent JSONL transcript parsing from projects/
    MessageService              # JSONL tail-reading for session/subagent conversation messages
    SessionActivityService      # Activity status (thinking/waiting/idle/error) + token usage (incl. latest-turn context size) from JSONL
    ToolStatsService            # Per-session tool-call aggregation (success/failed/rejected, output-impact) from JSONL
    UsageService                # Per-participant token/cost breakdown (lead session + each subagent), composes SessionActivityService + SubagentService
    ClaudeDirectoryWatcher      # BackgroundService with 4 FileSystemWatchers (extension-filtered)
    SseClientManager            # SSE client connection manager (singleton)
  UpdateCheck/
    Models/                     # NuGetVersionIndex, UpdateCheckCache (internal)
    Services/                   # UpdateCheckService (BackgroundService), logger messages
  wwwroot/
    index.html                  # Single-page Kanban + Timeline dashboard (embedded resource)
    images/icon.png             # ATC logo (favicon + sidebar)
  GlobalUsings.cs               # All using directives centralized here
test/Atc.Claude.Kanban.Tests/
  Helpers/                      # PathHelper, MockHttpMessageHandler
  Services/                     # SessionService, TaskService, TeamService, SubagentService,
                                # MessageService, SessionActivityService, PlanService,
                                # SseClientManager, UpdateCheckService, ToolStatsService, UsageService tests
scripts/
  seed-demo-fixture.ps1         # Materializes a synthetic ~/.claude tree for reproducible README screenshots (run with --dir)
```

## Architecture

- **ASP.NET Core Minimal APIs** with `Atc.Rest.MinimalApi` endpoint definitions
- **FileSystemWatcher** + `System.Threading.Channels` for event-driven file monitoring
- **Server-Sent Events** via `Results.Stream` with raw UTF-8 byte writes (NOT StreamWriter — Kestrel disallows synchronous Flush)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atc-net/atc-claude-kanban](https://github.com/atc-net/atc-claude-kanban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
