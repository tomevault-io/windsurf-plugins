---
trigger: always_on
description: AI SDK benchmarking tool built with Vercel AI SDK and Bun runtime. Tests AI agents with MCP (Model Context Protocol) server integration using the Vercel AI Gateway. Automatically discovers and runs all tests in the `tests/` directory and verifies LLM-generated Svelte components against test suites.
---

## Project Overview

AI SDK benchmarking tool built with Vercel AI SDK and Bun runtime. Tests AI agents with MCP (Model Context Protocol) server integration using the Vercel AI Gateway. Automatically discovers and runs all tests in the `tests/` directory and verifies LLM-generated Svelte components against test suites.

## Development Commands

```bash
# Install dependencies (runs patch-package automatically)
bun install

# Run the main benchmark (interactive CLI)
bun run start

# Verify reference implementations against test suites
bun run verify-tests

# Generate HTML reports from all result JSON files
bun run generate-report.ts

# Generate HTML report from specific result file
bun run generate-report.ts results/result-2024-12-07-14-30-45.json

# Generate index.html with all results
bun run generate-index

# Build all reports (generate-report + generate-index)
bun run build

# Run unit tests for lib modules
bun test

# Run specific test file
bun test lib/utils.test.ts

# Run TypeScript type checking
bun tsc --noEmit

# Format code with Prettier
bun run prettier

# Lint code
bun run lint

# Lint and fix code
bun run lint:fix

# Link to Vercel project for AI Gateway
bun run vercel:link

# Pull environment variables from Vercel
bun run vercel:env:pull
```

## Environment Variables

### Vercel AI Gateway

The benchmark uses the Vercel AI Gateway for model access. Configuration:

1. Link to a Vercel project with AI Gateway enabled: `bun run vercel:link`
2. Pull environment variables: `bun run vercel:env:pull`

Required environment variable:

- `VERCEL_OIDC_TOKEN`: OIDC token for Vercel AI Gateway authentication

### MCP Server Configuration

MCP integration is configured via the interactive CLI at runtime. Options:

- **No MCP Integration**: Agent runs with built-in tools only
- **MCP over HTTP**: Uses HTTP transport (default: `https://mcp.svelte.dev/mcp`)
- **MCP over StdIO**: Uses local command (default: `npx -y @sveltejs/mcp`)

## Architecture

### Directory Structure

```
├── index.ts                    # Main entry point with interactive CLI
├── lib/
│   ├── pricing.ts              # Cost calculation from gateway pricing
│   ├── pricing.test.ts         # Unit tests for pricing module
│   ├── test-discovery.ts       # Test suite discovery
│   ├── output-test-runner.ts   # Vitest runner for component verification
│   ├── output-test-runner.test.ts # Unit tests for output runner
│   ├── validator-runner.ts     # Optional validator runner
│   ├── validator-runner.test.ts # Unit tests for validator runner
│   ├── verify-references.ts    # Reference implementation verification
│   ├── report.ts               # Report generation orchestration
│   ├── report.test.ts          # Unit tests for report module
│   ├── report-template.ts      # HTML report template generation
│   ├── report-styles.ts        # CSS styles for HTML reports
│   ├── token-cache.ts          # Token cache simulation for cost estimation
│   ├── utils.ts                # Utility functions (sanitization, cost calculation, etc.)
│   ├── utils.test.ts           # Unit tests for utility functions
│   └── tools/
│       ├── index.ts            # Tool exports
│       ├── result-write.ts     # ResultWrite tool for final output
│       ├── result-write.test.ts # Unit tests for ResultWrite tool
│       ├── test-component.ts   # TestComponent tool for iterative testing
│       └── test-component.test.ts # Unit tests for TestComponent tool
├── tests/                      # Benchmark test suites
│   └── {test-name}/
│       ├── Reference.svelte    # Reference implementation (required)
│       ├── test.ts             # Vitest test file (required)
│       ├── prompt.md           # Agent prompt (required)
│       └── validator.ts        # Optional custom validator
├── results/                    # Benchmark results (JSON + HTML)
├── outputs/                    # Temporary directory for test verification
└── patches/                    # Patches for dependencies
```

### Test Suite Structure

Benchmark test suites in `tests/` directory:

```
tests/
  {test-name}/
    Reference.svelte  - Reference implementation of the component
    test.ts          - Vitest test file (imports "./Component.svelte")
    prompt.md        - Prompt for AI agents to implement the component
```

**Benchmark Workflow:**

1. `index.ts` presents interactive CLI for model/MCP selection
2. Discovers all test suites in `tests/`
3. For each selected model and test:
   - Loads `prompt.md` and builds agent prompt
   - Agent generates component code using available tools
   - Agent calls `ResultWrite` tool with the component code
   - Component is written to `outputs/{test-name}/Component.svelte`
   - Test file is copied to `outputs/{test-name}/test.ts`
   - Vitest runs tests against the generated component
   - Results are collected (pass/fail, error messages)
   - Output directory is cleaned up
4. All results are saved to timestamped JSON file
5. HTML report is generated with expandable sections for each test

### Agent Tools


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sveltejs/ai](https://github.com/sveltejs/ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
