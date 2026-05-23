---
trigger: always_on
description: This is a boilerplate template repository. When using this template:
---

# MCP Integration Guidelines

## Template Context

This is a boilerplate template repository. When using this template:

1. **Repository Initialization**: After creating from template, run `./scripts/init-template.sh` to replace all `{{REPO_NAME}}`, `{{GITHUB_USER}}`, and `{{REPO_PATH}}` placeholders.

2. **MCP Server Setup**: Copy `.mcp.example.json` to `claude.json` and configure:
   - kit-dev for codebase intelligence
   - Ref for documentation research
   - Linear for ticket management (optional)

3. **Thoughts Submodule**: Create a separate repository for your project's thoughts and add as submodule, or remove thoughts references if not needed.

---

This project uses Kit MCP (codebase intelligence), Ref MCP (documentation research), and Linear MCP (ticket management) to enhance code analysis and development workflow.

## Tool Selection Decision Tree

### For Local Codebase Work

**Use Kit MCP when:**
- Finding symbol definitions → `extract_symbols`
- Understanding code structure → `get_file_tree`, `extract_symbols`
- Tracing symbol usage → `find_symbol_usages`
- Tracing dependencies manually → `extract_symbols` + `find_symbol_usages`
- Searching code patterns → `grep_code` (literal) or `search_code` (pattern)

**Use built-in tools when:**
- Reading specific files → `Read` (faster, complete content)
- Searching non-code files → `Grep` (configs, markdown, JSON)
- File path patterns → `Glob` (finding files by name)
- One-off searches → `Grep` (no caching benefit from Kit MCP)

**Note:** Kit MCP tools work best with Python and TypeScript. For other languages (Go, Rust, etc.), built-in tools may be more reliable.

### For External Documentation

**Use Ref MCP when:**
- Researching APIs → `ref_search_documentation`
- Reading documentation → `ref_read_url`
- Finding integration guides → `ref_search_documentation`
- Debugging third-party packages → `ref_search_documentation`

**Use WebSearch when:**
- Current events or news (post-January 2025)
- Real-time data
- General information (non-technical)
- Documentation not in Ref's curated index

**Use WebFetch when:**
- Reading specific URLs not available via Ref MCP
- Fetching content from blogs, tutorials, or non-official sources

### For Linear Ticket Management

**Use Linear MCP when:**
- Reading ticket details → `linear-ticket-reader` agent
- Searching for related tickets → `linear-searcher` agent
- Updating ticket status or comments → Via `/linear` command
- Getting ticket context for implementation → `linear-ticket-reader` agent

**Use `/linear` command directly when:**
- Creating tickets from thoughts documents
- Adding comments with conversation context
- Manually updating ticket fields
- Managing ticket workflow progression

## Repository Initialization

At the start of codebase work:
1. `open_repository` to load context (enables Kit MCP caching)
2. `get_file_tree` to understand structure (Git-aware, respects .gitignore)
3. `extract_symbols` for fast symbol overview (results are cached)
4. Then proceed with focused analysis using appropriate tools

**Important:** Repository IDs (e.g., `repo_0`) reset between Claude Code sessions. Re-initialize when starting a new session.

## Large Repository Handling

### Known Limitation: Subdirectory .gitignore Support

Kit-dev's `get_file_tree` currently only loads the root `.gitignore` file. Repositories with subdirectory `.gitignore` files (common in monorepos) may return excessive file counts.

**Symptoms:**
- Token overflow errors (>25k tokens)
- File count much higher than `git ls-files`
- Includes node_modules, venv, or other ignored directories

**Workaround**: Use targeted tools instead of full tree:

```bash
# Instead of get_file_tree:
Glob("**/*.py")           # Find Python files
Grep("pattern", type="ts") # Search TypeScript
LS specific/directory/     # List specific paths
```

**When to use get_file_tree**:
- Small repos (<1000 files)
- Repos with complete root .gitignore
- After verifying file count with: `git ls-files | wc -l`

**When to use alternatives**:
- Large repos (>5000 files)
- Monorepos with multiple node_modules
- Repos with subdirectory .gitignore files

**Fix Status**: Upstream fix in progress (see `thoughts/shared/plans/2025-10-05-SOL-1-kit-mcp-gitignore-fix.md`)

## Agent Usage

Commands will automatically use MCP-enhanced agents when available:

- **`codebase-locator`**: Locates files and components using Grep/Glob/LS + Kit MCP AST tools
  - Use when: Finding where features are implemented, locating specific code

- **`codebase-analyzer`**: Analyzes implementation details using Read + Kit MCP symbol/dependency tools
  - Use when: Understanding how something works, analyzing code structure

- **`codebase-pattern-finder`**: Finds similar implementations using Grep/Read + Kit MCP pattern tools
  - Use when: Looking for examples, finding existing patterns to follow

- **`external-doc-researcher`**: Researches external documentation using Ref MCP + WebSearch
  - Use when: Researching frameworks, APIs, best practices from official docs

- **`mcp-package-researcher`**: Deep package research using Ref MCP (docs) + Kit MCP package search (source)
  - Use when: Planning package integration, debugging third-party libraries, evaluating packages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ferymad/claudecodetest](https://github.com/Ferymad/claudecodetest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
