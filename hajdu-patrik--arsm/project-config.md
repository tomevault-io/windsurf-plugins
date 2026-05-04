---
trigger: always_on
description: > **Architecture Notice:** This project uses both GitHub Copilot and Claude Code as the primary agentic AI tools. To maintain consistency across the workspace, ensure that any architectural rules or domain constraints updated in this file are also synchronized with the `CLAUDE.md` and `.claude/skills/` files.
---

> **Architecture Notice:** This project uses both GitHub Copilot and Claude Code as the primary agentic AI tools. To maintain consistency across the workspace, ensure that any architectural rules or domain constraints updated in this file are also synchronized with the `CLAUDE.md` and `.claude/skills/` files.

# ARSM (AutoService) Copilot Instructions (Project-Specific)

## Goal
This repository hosts the **ARSM** (Appointment and Resource Scheduling Management) full-stack application — a mechanic-facing workshop management tool for auto service businesses.

Prioritize maintainable, domain-safe, incremental changes that align with the existing architecture and folder layout.

## Technology Baseline
- Backend: .NET 10 (C# 15) ASP.NET Core Web API + Entity Framework Core.
- Frontend: React 19 + TypeScript + Vite.
- Styling: Tailwind CSS only.
- Orchestration: .NET Aspire (`AutoService.AppHost` + `AutoService.ServiceDefaults`).
- Database target: PostgreSQL via Aspire orchestration.

## Repository Map
- `app/AutoService.ApiService`: API, domain model, EF Core context and migrations.
- `app/AutoService.AppHost`: Aspire orchestration entry point.
- `app/AutoService.ServiceDefaults`: shared defaults and cross-service settings.
- `app/AutoService.WebUI`: React client.

## Team Coordination Rule (Merge-Conflict Prevention)
- If someone starts working in a shared or high-churn area, they should post a short note in the team group first (scope + expected files).
- For parallel work, prefer folder-level ownership during a work window (for example, one person on `ApiService/Auth`, another on `WebUI/src`).
- Before pushing larger changes, sync in the group to avoid simultaneous edits on the same files.

## AI SQL Safety Rule (Mandatory)
- For AI-assisted DB checks, use `ai_agent_test_user` only.
- Restrict AI SQL execution to read-only `SELECT` queries.
- Never execute DML/DDL from AI SQL tools (`INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `ALTER`, `CREATE`, `DROP`, `GRANT`, `REVOKE`).

## Documentation Sync Rule (Mandatory)
- After any change that affects API endpoints, EF migrations, middleware pipeline, WebUI pages/components/routes, dependencies (NuGet or npm), AppHost resource wiring, or configuration keys — run `/docs-sync` before considering the task complete.
- This keeps all `CLAUDE.md` files and `.github/instructions/` files in sync with the actual code.
- Trigger sections: endpoints, migrations, middleware order, pages, components, routes, stores, services, dependencies, config keys, AppHost resources, security settings (lockout, rate limits, token lifetimes).

## Code Documentation Style Rule (Mandatory)
- When adding or changing non-trivial classes/methods, use JSDoc-style block comments.
- Do not use XML documentation comments (`/// <summary>`, `/// <param>`, `/// <returns>`).
- Use the `coding-principles` agent after code changes that introduce/modify classes or methods.

## Conditional Test Execution Rule (Mandatory)
- Development default: for non-behavioral changes (for example refactor, naming, comments, formatting, docs-only updates, or internal restructuring without contract/flow changes), do **not** run `http-endpoint-test`, `sql-database-test`, or `e2e-playwright-test`.
- Run test skills when behavior changes or a new feature is introduced that affects API/UI/schema behavior.
- Always honor explicit user requests: if the prompt directly asks to run tests or create/update test suites, run the requested test agents regardless of change size.
- If there is no behavior/feature change and no explicit test request, run `docs-sync` only for the test-skill layer.
- When both API and schema behavior changed, `http-endpoint-test` and `sql-database-test` remain parallelizable.

## MCP Policy (Workspace)
- Keep MCP server setup intentionally minimal and project-focused.
- Track MCP config via templates: `.vscode/mcp.template.json` and `.claude/.mcp.template.json`.
- Runtime MCP files stay local/ignored: `.vscode/mcp.json` and `.claude/.mcp.json`.
- Keep `.vscode` and `.claude` MCP server sets aligned.
- Current shared server set:
	- `context-mode`
	- `aspire` (workspace-local tool via `dotnet tool run aspire -- mcp start`)
	- `postgres`
	- `docker`
- Local tool manifest for Aspire CLI: `dotnet-tools.json` at repository root.
- Default workflow: treat context-mode as automatic routing/enforcement. Do not require explicit context-mode prompts for routine small tasks.
- Prefer explicit context-mode tool usage when output can be large (long logs, broad searches, large API/CLI output, large docs/web content).
- For multi-step research, prefer batching/indexing patterns (`ctx_batch_execute`, indexing + search) over many separate high-output calls.
- After editing MCP templates or runtime MCP config, restart VS Code to ensure routing instructions are reloaded.

## Specialist Agents (`.github/agents/`, Mandatory Delegation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hajdu-patrik/ARSM](https://github.com/hajdu-patrik/ARSM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
