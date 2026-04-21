---
trigger: always_on
description: This file provides essential guidance to Claude Code when working with the xclaude-plugin repository.
---

# CLAUDE.md - AI Assistant Context

This file provides essential guidance to Claude Code when working with the xclaude-plugin repository.

## Project Overview

**xclaude-plugin** is a production-ready MCP (Model Context Protocol) plugin for iOS development automation. It provides 8 modular, workflow-specific MCP servers that consolidate 23 iOS operations across build, test, simulator control, and UI automation.

**Status**: ✅ Production-ready (v0.4.0) - All operations fully implemented, tested, and with codecov integration.

**Key Features**:

- **8 modular MCP servers** - Workflow-specific, composable architecture
- **23 total operations** across Xcode, Simulator, and IDB domains
- **8 procedural Skills** (on-demand documentation with examples)
- **100% TypeScript** with zero tolerance for `any` types
- **Full test coverage** with codecov reporting

## Repository Structure

```
xclaude-plugin/
├── README.md                      # User-facing plugin documentation
├── CLAUDE.md                      # This file - AI assistant context
├── CODESTYLE.md                   # Development guidelines
├── .claude-plugin/                # Plugin configuration
│   ├── plugin.json               # Plugin manifest (entry point)
│   └── marketplace.json          # Marketplace configuration
├── mcp-servers/                   # 8 modular MCP servers
│   ├── xc-build/                 # Build, clean, list schemes
│   ├── xc-launch/                # Simulator lifecycle: install and launch
│   ├── xc-interact/              # Simulator + IDB UI automation
│   ├── xc-ai-assist/             # AI-specific workflows
│   ├── xc-setup/                 # Health check, project info
│   ├── xc-testing/               # Test execution, results parsing
│   ├── xc-meta/                  # Metadata and introspection
│   └── xc-all/                   # All-in-one consolidated server
│   └── [each server]/
│       ├── src/
│       │   ├── index.ts          # Server entry point
│       │   └── operations/       # Operation implementations
│       ├── package.json
│       ├── tsconfig.json
│       └── dist/                 # Compiled JavaScript (committed for distribution)
├── skills/                        # 8 procedural Skills (on-demand)
│   ├── xcode-workflows/
│   ├── simulator-workflows/
│   ├── ui-automation-workflows/
│   ├── accessibility-testing/
│   ├── ios-testing-patterns/
│   ├── crash-debugging/
│   ├── performance-profiling/
│   └── state-management/
├── .mcp.json                      # Default MCP configuration (loads xc-setup, xc-build)
├── .mcp.json.example              # Full configuration showing all 8 servers
└── .gitignore                     # Version control exclusions
```

## Default vs Optional Servers

**Default Load** (in `.mcp.json`):

- `xc-setup` - Environment validation, project introspection
- `xc-build` - Build, clean, list schemes

**Optional Load** (documented in README):

- `xc-launch` - Simulator lifecycle: install and launch app
- `xc-testing` - Test execution and result analysis
- `xc-interact` - Simulator and IDB UI automation
- `xc-ai-assist` - AI-specific workflows
- `xc-meta` - Metadata and introspection
- `xc-all` - All 24 operations in one server

Users can customize `.mcp.json` to load any combination of servers based on their workflow.

## Code Style Principles

**IMPORTANT**: Always follow [CODESTYLE.md](./CODESTYLE.md) when writing or modifying code.

Key principles across all servers:

- **Zero tolerance for `any`** - All types explicitly defined (MCP SDK constraint exception documented)
- **No silent failures** - Always handle and log errors
- **Constants over magic numbers** - All config in constants.ts per server
- **Progressive disclosure** - Document essentials, details on-demand
- **Function size** - Prefer <50 lines, maximum 100 lines
- **Security first** - Use spawn (not shell) for command execution
- **Comprehensive JSDoc** - All public APIs must be documented
- **Consistent error handling** - Structured error responses

Pre-commit validation:

```bash
npm run build          # TypeScript compilation for all servers
npm run test           # Run test suites for all servers
npm run coverage       # Generate coverage reports
```

## When Adding/Modifying Operations

1. **Add to specific server** - Choose appropriate server (or add to xc-all if generic)
2. **Implement operation** - Add to `src/operations/` with JSDoc
3. **Define types** - Update types in same file or shared types
4. **Add tests** - Create matching `.test.ts` file with full coverage
5. **Update README** - Document in server-specific README
6. **Build & validate** - Ensure `npm run build && npm run test` passes

## Development Commands

```bash
# Build all servers
npm run build              # TypeScript compilation

# Testing
npm run test               # Run all tests
npm run coverage           # Generate coverage reports

# Individual server commands (from root)
npm run build --workspace=xc-build      # Build specific server
npm run test --workspace=xc-build       # Test specific server

# Development
npm run clean              # Remove all dist/ directories
```

## Testing Strategy

All servers include:

- Unit tests for each operation
- Integration tests for workflows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [conorluddy/xclaude-plugin](https://github.com/conorluddy/xclaude-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
