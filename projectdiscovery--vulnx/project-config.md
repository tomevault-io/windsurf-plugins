---
trigger: always_on
description: This is a Go-based CLI tool for navigating CVE (Common Vulnerabilities and Exposures) data, developed by ProjectDiscovery. The tool provides structured access to vulnerability databases with filtering, searching, and mapping capabilities.
---

# vulnx Cursor Rules
# ProjectDiscovery CVE Navigation Tool - Go CLI Application

## Project Overview
This is a Go-based CLI tool for navigating CVE (Common Vulnerabilities and Exposures) data, developed by ProjectDiscovery. The tool provides structured access to vulnerability databases with filtering, searching, and mapping capabilities.

## Language & Framework
- **Language**: Go 1.22+
- **CLI Framework**: goflags (ProjectDiscovery)
- **HTTP Client**: retryablehttp-go (ProjectDiscovery)
- **Logging**: gologger (ProjectDiscovery)
- **Table Rendering**: go-pretty/v6
- **JSON Handling**: Standard library with extensive struct tags

## Project Structure
```
cmd/vulnx/          # Main application entry point
pkg/runner/          # Core application logic, CLI handling, and table rendering
pkg/service/         # API service layer for CVE data retrieval
pkg/types/           # Type definitions and data structures
pkg/testutils/       # Testing utilities
static/              # Static assets (images, etc.)
```

## MCP Tool Integration & Workflow

### Tool Priority & Philosophy
- **ALWAYS prioritize MCP tools over manual operations** - These tools provide enhanced capabilities and should be your first choice for any development task
- Use MCP tools as the primary method for code exploration, modification, testing, and documentation
- Combine multiple tools strategically to create efficient workflows
- Fall back to manual methods only when MCP tools are unavailable or insufficient

### Core Development Workflow Tools

#### Code Exploration & Analysis
- **`codebase_search`**: Use for semantic code searches to understand patterns, find similar implementations, and locate relevant code by meaning rather than exact text
- **`read_file`**: Primary method for examining file contents - always prefer this over manual file opening
- **`list_dir`**: Use for exploring project structure and understanding directory organization
- **`file_search`**: Employ fuzzy file path searches when you need to locate files without knowing exact paths
- **`grep_search`**: Perform exact text or regex searches across the codebase for precise pattern matching

#### Code Modification & Management
- **`edit_file`**: Primary tool for all code modifications, file creation, and content updates
- **`delete_file`**: Use for file removal operations instead of manual deletion
- **`reapply`**: Leverage for repeating similar edits across multiple files efficiently
- **`run_terminal_cmd`**: Execute all build, test, deployment, and command-line operations through this tool

### Knowledge Management & Documentation Tools

#### Information Gathering
- **`web_search`** and **`mcp_MCP_DOCKER_web_search_exa`**: Use for real-time information lookup, latest documentation, and current best practices
- **`get-library-docs`**: Fetch up-to-date library documentation instead of relying on potentially outdated local docs
- **`resolve-library-id`**: Identify and resolve package names to proper library identifiers
- **`fetch_pull_request`**: Look up PRs, issues, and commits for context and historical information

#### Documentation & Visualization
- **`create_diagram`**: Create Mermaid diagrams to visualize system architecture, data flows, and relationships
- **`update_memory`**: Maintain persistent knowledge base for project insights, decisions, and patterns
- **`todo_write`**: Create and manage structured task lists for complex development workflows

### Advanced Analysis & Planning Tools

#### Problem Solving & Planning
- **`sequentialthinking`**: Use for complex problem-solving, detailed analysis, and step-by-step reasoning before implementing solutions
- **Knowledge Graph Tools**: Leverage `create_entities`, `create_relations`, `search_nodes`, `read_graph`, etc. for:
  - Mapping complex relationships between code components
  - Understanding system dependencies and interactions
  - Documenting architectural decisions and patterns
  - Managing complex data structures and their relationships

### Tool Integration Patterns

#### Recommended Workflows
1. **Code Investigation**: `codebase_search` → `read_file` → `grep_search` (if needed) → `sequentialthinking` (for analysis)
2. **Feature Implementation**: `sequentialthinking` (planning) → `codebase_search` (understanding existing patterns) → `edit_file` → `run_terminal_cmd` (testing)
3. **Bug Fixing**: `grep_search` (find issue) → `read_file` (understand context) → `codebase_search` (find related code) → `edit_file` (fix) → `run_terminal_cmd` (verify)
4. **Documentation**: `read_file` → `create_diagram` → `update_memory` → `edit_file` (update docs)
5. **Research & Learning**: `web_search` → `get-library-docs` → `update_memory` → `create_diagram` (if complex)

#### Best Practices
- Start complex tasks with `sequentialthinking` to break down the problem
- Use `codebase_search` to understand existing patterns before implementing new features
- Combine `create_diagram` with `update_memory` to document architectural decisions
- Leverage knowledge graph tools for managing complex system relationships

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectdiscovery/vulnx](https://github.com/projectdiscovery/vulnx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
