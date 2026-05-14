---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Visor is an AI-powered code review tool for GitHub Pull Requests that can run as both a GitHub Action and CLI tool. The project analyzes code for security, performance, style, and architectural issues using various AI providers (Google Gemini, Anthropic Claude, OpenAI GPT) and provides intelligent feedback.

## Development Commands

### Build and Development
- `npm run build` - Build TypeScript to dist/
- `npm run clean` - Remove dist/ directory
- `npm run prebuild` - Runs clean before build

### Testing
- `npm test` - Run all Jest tests (timeout: 10 minutes per user config)
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Generate coverage report

### Code Quality
- `npm run lint` - Lint TypeScript files in src/ and tests/
- `npm run lint:fix` - Auto-fix linting issues
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check code formatting

### Running the Tool
- `./dist/cli-main.js --help` - Show CLI help after building
- `./dist/cli-main.js --check all` - Run all checks
- `./dist/cli-main.js --check security --output json` - Security check with JSON output
- `./dist/cli-main.js config snapshots` - List config snapshots
- `./dist/cli-main.js --slack --config .visor.yaml --watch` - Slack mode with live config reload

### Deployment
- `npm run deploy:site` - Deploy site to Cloudflare Pages
- `npm run deploy:worker` - Deploy worker to Cloudflare
- `npm run deploy` - Deploy both site and worker

## Architecture Overview

### Core Components

**Entry Points:**
- `src/index.ts` - GitHub Action entry point, handles GitHub event processing
- `src/cli-main.ts` - CLI tool entry point, parses arguments and runs checks

**Key Services:**
- `src/reviewer.ts` - Core review logic and issue scoring calculations
- `src/ai-review-service.ts` - AI provider abstraction for code analysis
- `src/pr-analyzer.ts` - GitHub PR data extraction and diff processing
- `src/pr-detector.ts` - Detects PR context from various GitHub events
- `src/github-comments.ts` - Manages GitHub PR comments and updates
- `src/check-execution-engine.ts` - Orchestrates running multiple check providers

**Provider System:**
- `src/providers/` - Pluggable check provider architecture
  - `ai-check-provider.ts` - AI-powered analysis (Gemini, Claude, OpenAI)
  - `mcp-check-provider.ts` - Direct MCP tool execution via stdio/SSE/HTTP
  - `utcp-check-provider.ts` - UTCP tool execution via native protocols (HTTP/CLI/SSE)
  - `claude-code-check-provider.ts` - Claude Code SDK integration with MCP tools
  - `tool-check-provider.ts` - Integration with external tools
  - `command-check-provider.ts` - Execute shell commands
  - `http-check-provider.ts` - HTTP webhook output
  - `http-input-provider.ts` - HTTP webhook input
  - `http-client-provider.ts` - HTTP client for external APIs
  - `noop-check-provider.ts` - No-op provider for command orchestration
  - `check-provider-registry.ts` - Provider registration system
  - `mcp-tools.ts` - MCP tool definitions and server management
  - `claude-code-types.ts` - TypeScript types for Claude Code SDK

**Configuration:**
- `src/config.ts` - Configuration loading and management
- `src/types/config.ts` - TypeScript types for configuration
- Default config file: `.visor.yaml` (not `visor.config.yaml`)

### Key Features

1. **Dual Operation Modes**: GitHub Action and CLI tool with shared core logic
2. **Pluggable Providers**: Extensible system for different analysis types
3. **AI Integration**: Multi-provider AI support including Claude Code SDK
4. **MCP Provider**: Direct MCP tool execution with stdio, SSE, and HTTP transports
5. **UTCP Provider**: Direct UTCP tool execution via native protocols (HTTP, CLI, SSE); also bridges to MCP for AI agent tool access via `ai_mcp_servers`
6. **Claude Code Provider**: Advanced AI with MCP tools, subagents, and streaming
6. **Incremental Analysis**: Smart PR updates that analyze only new commits
7. **Comment Management**: Unique comment IDs prevent duplicate reviews
8. **Multiple Output Formats**: table, json, markdown, sarif
9. **MCP Protocol Support**: Custom tools and external server integration

### Configuration System

The tool looks for configuration in this order:
1. CLI `--config` parameter
2. `.visor.yaml` in project root
3. Default configuration

Configuration supports:
- Project metadata and language detection
- File include/exclude patterns
- Check-specific settings and thresholds
- AI provider configuration
- Output formatting options

### Testing Structure

- `tests/unit/` - Unit tests for individual components
- `tests/integration/` - Integration tests with mocked GitHub API
- `tests/e2e/` - End-to-end tests with real GitHub scenarios
- `tests/fixtures/` - Test data and mock responses
- `tests/scenarios/` - Complex test scenarios
- `tests/performance/` - Performance and stress tests

### Important Implementation Notes

1. **AI Provider Fallback**: If no AI API key is configured, falls back to basic pattern matching
2. **XML Formatting**: Uses structured XML when sending data to AI providers for better analysis

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [probelabs/visor](https://github.com/probelabs/visor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
