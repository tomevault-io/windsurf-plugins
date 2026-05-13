---
trigger: always_on
description: This is a **Model Context Protocol (MCP) server** that provides AI-powered architectural analysis and ADR (Architectural Decision Record) management. The server integrates with AI assistants (Claude, Cline, Cursor) via the MCP protocol and uses OpenRouter.ai for generating actual analysis results instead of prompts.
---

# MCP ADR Analysis Server - AI Coding Agent Instructions

## Project Overview

This is a **Model Context Protocol (MCP) server** that provides AI-powered architectural analysis and ADR (Architectural Decision Record) management. The server integrates with AI assistants (Claude, Cline, Cursor) via the MCP protocol and uses OpenRouter.ai for generating actual analysis results instead of prompts.

**Core Architecture**: MCP server (`src/index.ts`) exposes 73 tools through `@modelcontextprotocol/sdk` that call into utilities for AI execution, caching, knowledge graphs, and memory management.

## Critical Technical Conventions

### ESM-Only Module System

- **Pure ESM** (no CommonJS): All imports must use `.js` extensions even for TypeScript files
- Use `import.meta.url` (never `__dirname`). Helper: `getCurrentDirCompat()` from `src/utils/directory-compat.ts`
- Example: `import { foo } from './bar.js'` (note `.js` extension)
- `package.json` declares `"type": "module"` and targets ES2022

### TypeScript Configuration

- **Strict mode enabled**: All strict type checks are enforced (see `tsconfig.json`)
- Output directory: `dist/src/` with source maps and declarations
- Tree-sitter native modules are mocked in Jest (`tests/__mocks__/`)

### Testing Requirements

- **Coverage threshold: 85%** (branches, functions, lines, statements)
- Jest 30+ with ESM support (`workerThreads: false` to allow dynamic imports)
- Run tests with: `npm test` or `make test`
- Tree-sitter modules must be mocked for Jest VM environment

## Development Workflows

### Build & Run

```bash
npm run build      # Clean + TypeScript compile to dist/
npm start          # Run production server from dist/
npm run dev        # Development server with tsx hot-reload
make ci            # Full pipeline: security, lint, test, build
```

### Testing Strategy

```bash
npm test                      # All tests with Jest
npm run test:coverage         # Generate coverage report
npm run test:unit             # Unit tests via scripts/test-infrastructure.sh
npm run test:integration      # Integration tests
npm run test:mcp-interactive  # Test with MCP Inspector
```

### Code Quality

- **Linting**: `npm run lint` runs ESLint + TypeScript type checking (`tsc --noEmit`)
- **Formatting**: Prettier is integrated; use `npm run format` or `npm run lint:fix`
- Pre-commit hooks enforce quality via Husky

## AI Execution Architecture

### Two-Mode Operation

1. **Full Mode** (with `OPENROUTER_API_KEY`): Returns actual AI-generated analysis results
2. **Prompt Mode** (fallback): Returns prompts for manual execution

**Key Files**:

- `src/utils/ai-executor.ts`: OpenRouter.ai integration with caching and retry logic
- `src/config/ai-config.ts`: AI configuration management
- Environment: Set `EXECUTION_MODE=full` and `OPENROUTER_API_KEY` for AI results

### Tool Response Pattern

All tools in `src/tools/` follow this structure:

```typescript
return {
  content: [{ type: "text", text: "result" }],
  isError?: boolean
};
```

## Memory & State Management

### Knowledge Graph System (`src/utils/knowledge-graph-manager.ts`)

- **Persistent storage** in OS temp directory: `$TMPDIR/{projectName}/cache/`
- Tracks intents, tool executions, ADR decisions, and relationships
- Provides `queryKnowledgeGraph()` for semantic context retrieval
- Memory operations are recorded for analytics (limited to last 1000 entries)

### State Reinforcement (`src/utils/state-reinforcement-manager.ts`)

- **Context decay mitigation**: Re-injects core context every 5 turns or when responses exceed 3000 tokens
- Integrates recent knowledge graph intents into context reminders
- Configuration: `turnInterval`, `tokenThreshold`, `includeKnowledgeGraphContext`

### Conversation Memory (`src/utils/conversation-memory-manager.ts`)

- **Phase 3 context decay mitigation**: Structured external memory for long conversations
- Stores conversation sessions, expandable content, and resumption context
- Auto-cleanup of sessions older than 24 hours

## Deployment Pattern Framework

### Validated Patterns (`patterns/infrastructure/*.yaml`)

Authoritative templates for infrastructure deployment that LLMs query for platform-specific guidance:

**Available Patterns**:

- `kubernetes.yaml` - Container orchestration
- `firebase.yaml` - Production Firebase deployment
- `firebase-emulators.yaml` - Local Firebase testing (emulator-first workflow)
- `aws.yaml`, `openshift.yaml` - Cloud platforms

**Pattern Structure**:

- `authoritativeSources`: URLs with priority (1-10) for LLMs to query
- `deploymentPhases`: Ordered steps with commands
- `validationChecks`: Critical checks with remediation steps
- `detectionHints`: Files/patterns for automatic pattern detection (e.g., `firebase.json` → Firebase)

**Tool Integration**: `bootstrap-validation-loop-tool.ts` auto-detects patterns based on project files with confidence scoring.

## Key Tool Categories

### Architectural Analysis

- `analyze_project_ecosystem`: Comprehensive project analysis with technology detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tosin2013/mcp-adr-analysis-server](https://github.com/tosin2013/mcp-adr-analysis-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
