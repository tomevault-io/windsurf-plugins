---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FlowBook is a JupyterLab 4.0+ extension that combines a TypeScript frontend with a Python server extension and custom IPython kernels. The extension provides notebook analysis, validation, execution, reproducibility enforcement, and AI-powered capabilities through a command-based architecture.

## Development Commands

### Initial Setup

```bash
# Install package in development mode
pip install -e "."

# Link development version with JupyterLab
jupyter labextension develop . --overwrite

# Enable server extension
jupyter server extension enable flowbook
```

### Building

```bash
# Development build (with source maps)
jlpm build

# Production build
jlpm build:prod

# Clean build artifacts
jlpm clean:all
```

### Development Workflow

```bash
# Terminal 1: Auto-rebuild TypeScript on changes
jlpm watch

# Terminal 2: Run JupyterLab
jupyter lab
```

After making changes, refresh JupyterLab in the browser. The `jlpm watch` command automatically rebuilds TypeScript and the labextension.

### Linting

```bash
# Run all linters with auto-fix
jlpm lint

# Individual linters
jlpm eslint          # TypeScript/JavaScript
jlpm prettier        # Code formatting
jlpm stylelint       # CSS

# Check without fixing
jlpm lint:check
```

### Testing

```bash
# Run Python tests
pytest flowbook/

# Run specific test file
pytest flowbook/kernel/tests/test_reproducibility_enforcer.py
```

Test files (`test_*.py`) must be placed in a `tests/` subdirectory of the package they test. For example, tests for `flowbook/kernel/` go in `flowbook/kernel/tests/`. Each `tests/` directory must contain an `__init__.py` file.

### Verification

```bash
# Check server extension is enabled
jupyter server extension list

# Check frontend extension is installed
jupyter labextension list
```

## Architecture

### Four-Tier Structure

1. **Frontend (TypeScript)**: `src/` - JupyterLab UI plugin for reproducibility visualization
2. **Server Extension (Python)**: `flowbook/server/` - HTTP handlers and command processing
3. **MCP Server (Python)**: `flowbook/mcp/` - MCP tools for AI-driven notebook analysis, with real-time collaboration with JupyterLab
4. **Custom Kernels**: IPython kernels for different use cases
   - `flowbook/kernel/` - FlowBook kernel with always-on reproducibility tracking
   - `flowbook/checkpoint_kernel/` - Checkpoint kernel for timing/benchmarking

### Frontend Components (`src/`)

The frontend exports a single JupyterLab plugin that activates for `flowbook_kernel`:

```
src/
├── index.ts                 # Exports [flowbookPlugin]
├── api.ts                   # FlowbookAPI for HTTP communication
├── kernel.ts                # KernelUtils (startup, info)
├── handler.ts               # Request handler (ServerConnection wrapper)
├── cellindex.ts             # Cell index DOM overlay manager (@A notation)
├── cellindexutils.ts        # indexToAlpha/alphaToIndex + getCodeCellOrder utility
├── shared/
│   ├── kerneldetection.ts   # KernelDetector class
│   └── types.ts             # IKernelInfo, ICommandResult, IExecuteCommandRequest
├── flowbook/                # FlowBook kernel plugin (reproducibility tracking)
│   ├── plugin.ts            # Plugin activation, kernel discovery, activation manager
│   ├── types.ts             # IReproducibilityMetadata, IPredicateViolation, IStalenessReason
│   ├── protocol.ts          # FlowBook comm protocol types (shared with kernel)
│   ├── stalenessmanager.ts  # Tracks stale cells per notebook with signals
│   ├── stalenessnotice.ts   # Manages staleness notice outputs in cell output areas
│   ├── violationnotice.ts   # Manages violation notice outputs in cell output areas
│   ├── cellhighlighter.ts   # CSS highlighting + coordination (delegates to notice managers)
│   ├── executionhook.ts     # Comm-based kernel communication + cell edit detection
│   ├── toolbar.ts           # "Run Next Stale/Unrun" button
│   ├── metadatapanel.tsx    # Reproducibility metadata panel (reads, writes, stale cells)
│   └── dependenciespanel.tsx # ReactFlow dependency DAG visualization
└── _archived/               # Mothballed experimental plugin (excluded from build)
    ├── experimental/        # AI commands plugin (11 files)
    ├── logpanel.tsx         # SSE event stream display
    ├── executiondialog.tsx  # Modal dialog with real-time command output
    └── messagecomponents.tsx # Shared React message formatting
```

**Plugin Activation**: `flowbook:plugin` activates UI only when kernel is `flowbook_kernel`. On activation, writes a kernel discovery file so MCP can find and share the kernel.

### Server Extension (`flowbook/server/`)

The server uses the modern **ExtensionApp** pattern (not legacy extension points).

- `__init__.py` - `FlowBookExtension(ExtensionApp)` class with `initialize_handlers()` method
- `handlers.py` - HTTP request handlers:
  - `POST /flowbook/execute` - Execute a command (FlowbookCommandHandler)
  - `GET /flowbook/list` - List available commands (CommandListHandler)
  - `GET/PUT /flowbook/kernel-discovery/{path}` - Kernel discovery for MCP↔JupyterLab sharing (KernelDiscoveryHandler)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stephenfreund/FlowBook](https://github.com/stephenfreund/FlowBook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
