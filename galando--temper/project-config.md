---
trigger: always_on
description: Temper reference: plan
---



# Plan: Feature Planning with Impact Analysis

**Goal:** Transform feature request into implementation plan with blast radius analysis and risk assessment.

## Active Skills

- **Context Engineering** — load hierarchical context at stage start (rules → arch → source → errors, under 2K lines/task)
- **Temper Core** — stack detection, pack resolution, quality gates

## Usage

```
/temper:plan "feature description"
/temper:plan "JIRA-123"
/temper:plan "#123"
/temper:plan --full "feature"    # Force full spec-kit even for simple tasks
/temper:plan --quick "feature"   # Force lightweight plan
/temper:plan --reindex "feature" # Force full semantic index rebuild
```

## Feature: $ARGUMENTS

## Execution

### Context Loading

This stage may run in two modes:
- **Standalone** (`/temper:plan`) — runs in current context, handles its own gate
- **Agent subprocess** (from `/temper`) — runs in clean context, returns summary to orchestrator

**Subprocess mode override:** When running as an Agent subprocess, do NOT show AskUserQuestion gates or clear context. Return the plan summary to the orchestrator. The orchestrator handles all gate decisions and context transitions.

In both modes, the planning methodology is identical.

**Context loading strategy:** Apply the context-engineering skill for hierarchical loading (rules -> arch -> source -> errors, under 2K lines/task). The file list below specifies WHAT to load; the skill specifies HOW and WHEN.

### Phase 0: Detect Input Type

Determine what the user provided:

- Starts with a project prefix + numbers (e.g., `BKNG-1234`, `PROJ-567`) → fetch from Jira API using `gh` or `curl`
- Starts with `#` + numbers (e.g., `#1234`) → fetch from GitHub Issues using `gh issue view`
- URL containing `github.com` → fetch from GitHub using `gh`
- URL containing `atlassian.net` or `jira` → fetch from Jira API
- Everything else → use as direct feature description

**If issue tracker fetch fails:** Fall back to using the raw text as description. Never block on API failures.

### Phase 0.5: Initial File Count Estimate

Before launching the Explore subagent, make a rough estimate to guide planning depth:

```
ESTIMATION HEURISTIC:

| Feature Type | Typical Files | Complexity |
|-------------|---------------|------------|
| Single endpoint/page/handler | 3-5 | Simple |
| New domain/module | 7-12 | Medium-Complex |
| Cross-cutting (auth, middleware) | 10-25 | Complex |
| External service integration | 5-8 | Medium |
```

Pre-estimate only. Explore subagent refines with actual codebase knowledge.

### Phase 1: Auto-Prime (via Explore subagent)

Launch an Explore subagent with this prompt:

```
Scan this project to build a reference map for planning a new feature.

0. CODE REVIEW GRAPH MCP (PRIMARY exploration method)
   If code-review-graph MCP tools are available:
   a. Call build_or_update_graph_tool to ensure the graph is up-to-date
   b. Call get_architecture_overview_tool for high-level codebase structure
   c. Call list_communities_tool to understand code grouping
   d. Call list_flows_tool (limit 20) to identify key execution paths
   e. Call semantic_search_nodes_tool with keywords from the feature description
   f. Call get_minimal_context_tool with the feature description for a quick overview
   → Use these results as the PRIMARY source for dependency maps, patterns, and similar code
   → Still verify with direct file reads where needed
   If MCP is unavailable, proceed with grep/read steps below as fallback.

1. DETECT STACK
   - Look for: package.json, pom.xml, build.gradle, pyproject.toml, go.mod, Cargo.toml
   - Read the detected manifest to understand dependencies
   - Check for temper.config (stack override)

2. SCAN PROJECT STRUCTURE
   - List top-level directories
   - Identify source roots (src/, app/, lib/, etc.)
   - Count files by type to understand project scale

3. MAP PATTERNS
   For each layer found, note 1-2 example files:
   - Controllers/routes (API layer)
   - Services (business logic)
   - Repositories/DAOs (data access)
   - Models/entities
   - DTOs/schemas
   - Tests (unit + integration)
   - Configuration files

4. FIND SIMILAR IMPLEMENTATIONS
   Search for existing code similar to the planned feature.
   First try semantic_search_nodes_tool with feature keywords (if MCP available).
   Then grep for keywords from the feature description.

5. CHECK FOR COMPANY PRESET
   - Read .claude/temper.config if exists
   - Read .claude/presets/*.yaml if exists
   - Read enabled pack rules from .claude/packs/

6. READ SEMANTIC INDEX (if exists)
   - Read .temper/index/modules.json for dependency graph
   - Read .temper/index/api-surface.json for API map
   - If index doesn't exist or is stale, build it:
     - Map all imports/requires across the project
     - List all API endpoints with their handler functions
     - List all test files and what they test

Return a reference map in this format (max 60 lines):

STACK: {detected stack}
PRESET: {preset name or "none"}
PACKS: {enabled packs}

PROJECT STRUCTURE:
  {directory tree, 2 levels deep}

PATTERNS:
  Controllers: {pattern + example file}
  Services: {pattern + example file}
  Repositories: {pattern + example file}
  Tests: {pattern + example file}

SIMILAR CODE:
  {1-3 similar implementations found, with file paths}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galando/temper](https://github.com/galando/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
