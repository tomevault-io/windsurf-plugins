---
trigger: always_on
description: Musoq is a SQL query engine that lowers SQL queries through logical and physical query plans, lowers physical plans into Execution IR, renders that IR into executable .NET code at runtime, and runs it over diverse data sources (files, git, APIs, etc.) with nearly 1000 built-in methods.
---

# Musoq: SQL Query Engine Development Guide

Musoq is a SQL query engine that lowers SQL queries through logical and physical query plans, lowers physical plans into Execution IR, renders that IR into executable .NET code at runtime, and runs it over diverse data sources (files, git, APIs, etc.) with nearly 1000 built-in methods.

**Always reference these instructions first** and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Mandatory Planning Rule

Every time you generate a plan for a task, the **penultimate step** (second-to-last) must always be:

> **Re-read `copilot-instructions.md` and verify that all created or modified code follows every rule defined in it** — especially the Code Quality & Maintainability Standards section. If any violations are found, fix them before marking the task complete.

This is non-negotiable. Delivering working code that violates project standards is not acceptable. The final step of any plan should be running the relevant tests; the step immediately before that is the compliance check against these instructions.

## What is Musoq?

**Core Concept**: Musoq transforms SQL queries through typed intermediate plans into compiled C# code that executes against arbitrary data sources. It's designed for developers who want SQL's declarative power for everyday scripting tasks (file processing, git analysis, data transformation) instead of writing throwaway scripts.

**Key Architecture**: SQL text → typed AST → logical query plan → query planning decisions/properties → physical query plan → Execution IR → generated C# code → compiled .NET assembly → execution

## How These Instruction Files Fit Together

Musoq's guidance is layered. Read the most specific file that applies, then fall back to the broader ones:

| File(s) | Role | Audience |
|---------|------|----------|
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | Standalone root guide with the full rule set | GitHub Copilot |
| [CLAUDE.md](CLAUDE.md) | Root guide for Claude Code; mirrors the Copilot guide and `@`-imports the rule modules | Claude Code |
| `.claude/rules/*.md` | Canonical rule modules (`architecture`, `code-quality`, `multi-session`, `troubleshooting`, `validation`); also auto-loaded as workspace instruction files | All agents in this workspace |
| `src/dotnet/<Project>/copilot-instructions.md` | Per-project deep dives (internal structure, key classes, workflows) | Anyone editing that project |
| [.claude/rules/architecture.md](.claude/rules/architecture.md) | Authoritative logical/physical planner and execution-target ownership reference | Anyone touching IR, planner, or renderer code |

When two files appear to disagree, the more specific one wins for its scope: per-project files override the root guide for that project, and `.claude/rules/architecture.md` is authoritative for planner/IR ownership.

### Where Does My Change Belong?

| I want to… | Start in | Notes |
|------------|----------|-------|
| Add or change a built-in SQL function | `Musoq.Plugins/Lib/` | One partial `LibraryBase` per category; see the Plugins per-project file |
| Change SQL lexing, parsing, or AST nodes | `Musoq.Parser` | Lexer is direct character scanning; AST nodes drive everything downstream |
| Add a data source or change the schema contract | `Musoq.Schema` | `ISchema`/`ISchemaProvider`; keep public source APIs stable |
| Change what a query *means* (relational semantics) | `Musoq.Evaluator/IR/Logical` | Logical plan, not strategy |
| Choose an execution strategy (join/aggregate/window/paging) | `Musoq.Evaluator/IR/Planning` + `IR/Physical` | Strategy decisions live in the planner, not the renderer |
| Change executable operations or runtime metadata | `Musoq.Evaluator/IR/Execution` | Lowering coordinators + Execution IR records |
| Change only generated C# syntax | `IR/Execution/Rendering` or `IR/CodeGeneration` | Faithful emission only; never invent strategy here |
| Change compilation orchestration or the public API | `Musoq.Converter` | `InstanceCreator` is the public entry point |
| Add or adjust a performance benchmark | `Musoq.Benchmarks` | Establish a baseline before optimizing |

## Working Effectively

### Prerequisites and Environment Setup
- **Required**: .NET 10.0.300 SDK or newer 10.0 feature band (pinned in [global.json](global.json) with `rollForward: latestFeature`)
- **Recommended**: Visual Studio or VS Code with C# extension
- **OS**: Works on Windows, Linux, and macOS
- **Package Management**: Packages are generated explicitly with `dotnet pack`; release package versions live in [scripts/Versions.props](scripts/Versions.props). Publishing is tag-driven only; see [RELEASING.md](RELEASING.md).

### Core Development Workflow
Bootstrap, build, and test the repository:
```bash
# 1. Initial setup - takes ~30 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
dotnet restore src/dotnet/Musoq.sln --nologo --verbosity quiet


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Puchaczov/Musoq](https://github.com/Puchaczov/Musoq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
