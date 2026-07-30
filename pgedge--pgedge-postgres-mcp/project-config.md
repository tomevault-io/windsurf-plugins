---
trigger: always_on
description: > Standing instructions for Claude Code when working on this project.
---

# Claude Standing Instructions

> Standing instructions for Claude Code when working on this project.

## Project Structure

The pgEdge Postgres MCP Server is a single Go project with an embedded
React web client:

- `/cmd` - Command-line entry points (server, CLI client, KB builder).

- `/internal` - Core Go packages (MCP server, tools, resources, auth, chat).

- `/web` - Web client application (React/JavaScript).

- `/docs` - Documentation in markdown format with lowercase filenames.

- `/examples` - Example configuration files and demos.

## Key Files

Reference these files for project context:

- `docs/changelog.md` - Notable changes by release.

- `mkdocs.yml` - Documentation site navigation.

- `Makefile` - Build and test commands.

- `.claude/CLAUDE.md` - This file; project guidelines.

## Sub-Agents

Specialized sub-agents in `/.claude/agents/` handle complex domain tasks.
Use sub-agents proactively to preserve context in the main conversation.

### When to Use Sub-Agents

**Proactively delegate to sub-agents** for:

- Any Go implementation task (use **golang-expert**)
- Any React/JavaScript implementation task (use **react-expert**)
- Complex domain questions requiring research
- Tasks that would consume significant context in the main conversation

Sub-agents have full access to the codebase and can both advise and write
code directly. Delegating implementation work preserves context in the main
conversation for coordination and higher-level decisions.

### Available Sub-Agents

**Implementation Agents** (can write code):

- **golang-expert** - Go development: features, bugs, architecture, review.

- **react-expert** - React/MUI development: components, features, bugs.

- **documentation-writer** - Documentation following project style guide.

**Advisory Agents** (research and recommend):

- **postgres-expert** - PostgreSQL administration, tuning, troubleshooting.

- **mcp-server-expert** - MCP protocol, tool implementation, debugging.

- **testing-framework-architect** - Test strategies for Go and React.

- **security-auditor** - Security review, vulnerability detection, OWASP.

- **code-reviewer** - Code quality, bug detection, anti-patterns.

- **codebase-navigator** - Finding code, tracing data flow, structure.

Each sub-agent has a knowledge base in `/.claude/<agent-name>/` containing
domain-specific patterns and project conventions.

## Task Workflow

Follow this workflow for implementation tasks:

1. Read relevant code before proposing changes.

2. Use sub-agents for complex domain questions.

3. Run `make test` before marking implementation complete.

4. Review security implications for auth, input handling, or query changes.

5. Update `docs/changelog.md` for user-facing changes.

## General

- Always kill the MCP server and Vite server when no longer needed if they
  have been started to test something.

## Documentation

### General Guidelines

- Place comprehensive documentation in `/docs`.

- Create an `index.md` as the entry point; link to this from the README.

- Wrap all markdown files at 79 characters or less.

- Use lowercase filenames for all files in `/docs`.

### Writing Style

- Use active voice.

- Write grammatically correct sentences between 7 and 20 words.

- Use semicolons to link related ideas or manage long sentences.

- Use articles (a, an, the) appropriately.

- Avoid ambiguous pronoun references; only use "it" when the referent is
  in the same sentence.

### Document Structure

- Use one first-level heading per file with multiple second-level headings.

- Limit third and fourth-level headings to prominent content only.

- Include an introductory sentence or paragraph after each heading.

- For Features or Overview sections, use the format: "The MCP Server
  includes the following features:" followed by a bulleted list.

### Lists

- Leave a blank line before the first item in any list or sub-list.

- Write each bullet as a complete sentence with articles.

- Do not bold bullet items.

- Use numbered lists only for sequential steps.

### Code Snippets

- Precede code with an explanatory sentence: "In the following example,
  the `command_name` command uses..."

- Use backticks for inline code: `SELECT * FROM table;`

- Use fenced code blocks with language tags for multi-line code:

  ```sql
  SELECT * FROM code;
  ```

- Format `stdio`, `stdin`, `stdout`, and `stderr` in backticks.

- Capitalise SQL keywords; use lowercase for variables.

### Links and References

- Link files outside `/docs` to their GitHub location.

- Include third-party installation/documentation links in Prerequisites.

- Link to the GitHub repo when referencing cloning or project work.

- Do not link to github.io.

### README.md Files

At the top of each README:

- GitHub Action badges for repository actions.

- Table of Contents mirroring the `mkdocs.yml` nav section.

- Link to online docs at docs.pgedge.com.

README body content:

- Getting started steps.

- Prerequisites with commands and third-party links.

- Build/install commands and minimal configuration notes.

At the end of each README:

- Issues link: "To report an issue with the software, visit:"

- Online docs link: "For more information, visit
  [docs.pgedge.com](https://docs.pgedge.com)"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgEdge/pgedge-postgres-mcp](https://github.com/pgEdge/pgedge-postgres-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
