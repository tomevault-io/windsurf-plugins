---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this DiPeO repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this DiPeO repository.

## Project Overview

DiPeO is a monorepo for building and executing AI-powered agent workflows through visual programming:
- **Frontend** (apps/web/): React-based visual diagram editor
- **Server** (server/): FastAPI server with GraphQL API
- **CLI** (cli/): Command-line tools for running diagrams (`dipeo` and `dipeocc` commands)
- **Core Library** (dipeo/): Shared domain logic, execution engine, and infrastructure

## Essential Commands

### Setup & Development
```bash
make install          # Install dependencies (auto-installs uv)
make dev-all          # Start frontend + backend
make dev-web          # Frontend only (port 3000)
make dev-server       # Backend only (port 8000)
```

### Running Diagrams
```bash
# Synchronous execution (wait for completion)
dipeo run examples/simple_diagrams/simple_iter --light --debug --timeout=40 --timing # simple_iter_cc for claude code adapter diagram
dipeo run [diagram] --input-data '{"key": "value"}' --light --debug

# Asynchronous execution (background)
dipeo run examples/simple_diagrams/simple_iter --light --background --timeout=40
# Output: {"session_id": "exec_...", "status": "started"}

# Check execution status and results
dipeo results exec_9ebb3df7180a4a7383079680c28c6028
# Output: Full execution status with results, LLM usage, etc.

# Profile executions
dipeo metrics --latest --breakdown  # Profile latest execution
```

### Export to Python
```bash
dipeo export <diagram> <output.py> --light  # Export diagram to standalone Python script
dipeo export examples/simple_diagrams/simple_iter.light.yaml output.py --light
```
See [Diagram-to-Python Export Guide](docs/features/diagram-to-python-export.md)

### Compile and Push (for MCP)
```bash
dipeo compile my_diagram.light.yaml --light              # Validate
dipeo compile --stdin --light --push-as my_workflow      # Validate + push to MCP
# Also supports: --target-dir for custom location
```
See [MCP Server Integration](docs/features/mcp-server-integration.md#3-uploading-diagrams-for-mcp-access)

### Code Generation
```bash
cd dipeo/models && pnpm build      # Build TypeScript specs
make codegen                        # Generate code (→ staged/)
make diff-staged                    # Review changes
make apply-test                     # Apply with validation
make graphql-schema                 # Update GraphQL types
```

See [Code Generation Guide](docs/projects/code-generation-guide.md)

### DiPeOCC (Claude Code Session Conversion)
```bash
dipeocc list                    # List recent sessions
dipeocc convert --latest        # Convert latest
dipeocc convert {session-id}    # Convert specific
```
See [DiPeOCC Guide](docs/projects/dipeocc-guide.md)

### MCP Server (Expose DiPeO diagrams as MCP tools)
```bash
make dev-server                 # Start DiPeO server (port 8000)
ngrok http 8000                 # Expose via HTTPS (requires ngrok auth token)
# Access: https://your-url.ngrok-free.app/mcp/info
```
See [MCP Server Integration](docs/features/mcp-server-integration.md)

### Quality
```bash
make lint-server        # Lint Python
make lint-web           # Lint TypeScript
pnpm typecheck          # TypeScript checking
make format             # Format Python
```

### Documentation
```bash
make schema-docs        # Generate database schema docs
# Generates docs/database-schema.md (markdown) and docs/database-schema.sql (SQL reference)

python scripts/generate_light_diagram_schema.py  # Generate JSON Schema for light diagram format
# Reads auto-generated node schemas from diagram_generated/schemas/nodes/
```

## Working with Specialized Agents

**IMPORTANT**: Delegate domain-specific work to specialized agents. They have deep context and expertise.

### When to Delegate

**Always delegate when:**
- Task touches multiple files in a domain (handlers, CLI commands, React components)
- Debugging domain-specific issues (execution failures, codegen errors, type errors)
- Implementing new features in a domain (new node types, CLI commands, UI components)
- Exploring unfamiliar code areas

**Handle directly only when:**
- Single-file trivial edits (fixing typos, updating comments)
- Reading/searching for specific information
- Running simple commands

### Available Agents (Run in Parallel)

**Core Development:**
- **dipeo-package-maintainer** → /dipeo/ runtime (handlers, resolvers, EventBus, ServiceRegistry, domain models)
- **dipeo-backend** → /server/ + /cli/ (FastAPI server, CLI tools, database, MCP server)
- **dipeo-codegen-pipeline** → TypeScript specs → IR → Python/GraphQL generation
- **dipeo-frontend-dev** → React UI, visual editor, GraphQL hooks, type errors

**Specialized:**
- **codebase-auditor** → Security, performance, quality analysis
- **dipeocc-converter** → Claude Code session → DiPeO diagram conversion
- **code-polisher** → File separation, comment cleanup, import refactoring
- **codebase-qna** → Fast code search (functions, classes, patterns)

**Example**: `Task(dipeo-backend, "Add GraphQL timeout field") + Task(dipeo-frontend-dev, "Add timeout UI control")`

See [Agent docs](docs/agents/index.md) for detailed guides.

## Claude Code Skills


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sorryhyun/DiPeO](https://github.com/sorryhyun/DiPeO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
