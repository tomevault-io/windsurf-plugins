---
trigger: always_on
description: **Last Updated**: 2025-12-16
---

# AI Code Review - Claude AI Instructions

**Version**: 4.6.0
**Last Updated**: 2025-12-16
**Project Type**: CLI Tool / NPM Package
**Language**: TypeScript/JavaScript
**Package Manager**: pnpm (v8.15.0+)

## Priority Index

This index ranks instructions by priority for AI agents. Follow 🔴 Critical items first.

| Priority | Category | Description | Link |
|----------|----------|-------------|------|
| 🔴 | Security | API key handling and secret management | [Security Rules](#security-rules) |
| 🔴 | Build | Single-path build and test commands | [Build Commands](#build-commands) |
| 🔴 | Structure | File organization and naming standards | [Project Structure](#project-structure) |
| 🟡 | Development | Coding standards and patterns | [Development Guidelines](#development-guidelines) |
| 🟡 | Testing | Test requirements and commands | [Testing](#testing) |
| 🟡 | Quality | Linting and formatting standards | [Code Quality](#code-quality) |
| 🟢 | Release | Version management and publishing | [Release Process](#release-process) |
| 🟢 | Memory | KuzuMemory integration | [Memory Integration](#memory-integration) |
| ⚪ | Documentation | Documentation standards | [Documentation Standards](#documentation-standards) |

---

## Project Overview

AI Code Review is a TypeScript-based CLI tool for automated code reviews using multiple AI models:

- **Primary Models**: Google Gemini, Anthropic Claude, OpenAI GPT, OpenRouter
- **Purpose**: Automated code analysis with specialized review types
- **Architecture**: CLI tool with library exports for web integration
- **Package**: `@bobmatnyc/ai-code-review` on npm
- **Node Version**: >=20.0.0
- **Package Manager**: pnpm >=8.0.0

### Key Features

- **15+ Review Types**: comprehensive, quick-fixes, architectural, security, performance, unused-code, best-practices, evaluation, extract-patterns, coding-test, ai-integration, cloud-native, developer-experience, and more
- **Multi-Language Support**: TypeScript, JavaScript, Python, Ruby, PHP, Go, Java, Rust, Dart/Flutter
- **Semantic Chunking**: AI-guided TreeSitter-based code analysis (95%+ token reduction)
- **MCP Integration**: Model Context Protocol server for Claude Desktop
- **Library Mode**: Use as npm package in web applications
- **Interactive Mode**: Process review results in real-time

### Version 4.6.0 Release (2025-12-17)

This is a minor release with:
- API Key Validation on Startup with interactive recovery
- Project Configuration Storage in `.ai-code-review/config.yaml`
- Enhanced configuration precedence (CLI > Project Config > Env > Defaults)
- New `--skip-key-check` CLI flag
- Fixed model selection propagation throughout codebase

---

## Security Rules

### 🔴 CRITICAL: Never Commit API Keys

**ALWAYS follow these rules:**

1. **Environment Variables Only**: Use `.env.local` for API keys
2. **Check .gitignore**: Ensure `.env.local`, `tmp/`, `.claude-mpm/` are ignored
3. **Use Placeholders**: Example files must use `your_api_key_here`
4. **Review Before Commit**: Scan for secrets before `git add`
5. **Archive Location**: `docs/archive/` for sensitive docs (already in .gitignore)

### API Key Environment Variables

```bash
# Required: Model selection
AI_CODE_REVIEW_MODEL=gemini:gemini-2.5-pro

# Required: API key for selected provider
AI_CODE_REVIEW_GOOGLE_API_KEY=your_google_api_key_here
AI_CODE_REVIEW_ANTHROPIC_API_KEY=your_anthropic_api_key_here
AI_CODE_REVIEW_OPENROUTER_API_KEY=your_openrouter_api_key_here
AI_CODE_REVIEW_OPENAI_API_KEY=your_openai_api_key_here

# Optional: Separate model for consolidation/report writing
AI_CODE_REVIEW_WRITER_MODEL=openai:gpt-4o-mini
```

### Sensitive Files Checklist

- ❌ NEVER commit: `.env`, `.env.local`, `*.key`, `credentials.json`
- ✅ ALWAYS ignore: `tmp/`, `.claude-mpm/`, `docs/archive/`
- ✅ Example files: `.env.example` with placeholder values only

---

## Build Commands

### 🔴 Single-Path Workflow

**ONE command for each task:**

```bash
# Development
pnpm run dev              # Run from source with ts-node
pnpm run local            # Run with path resolution (tsconfig-paths)

# Build
pnpm run build            # Full build: test + types + bundle + sync
pnpm run quick-build      # Fast build: skip tests

# Testing
pnpm run test             # Run all tests (vitest)
pnpm run test:watch       # Watch mode
pnpm run test:coverage    # With coverage report
pnpm run test:e2e         # End-to-end tests

# Quality
pnpm run lint             # Check code with Biome
pnpm run lint:fix         # Auto-fix linting issues
pnpm run format           # Format code
pnpm run format:check     # Check formatting

# Release
pnpm run release:patch    # Bump patch version (1.0.x)
pnpm run release:minor    # Bump minor version (1.x.0)
pnpm run release:major    # Bump major version (x.0.0)
pnpm run release:dry-run  # Test release without publishing

# Verification
pnpm run ci:local         # Local CI checks (before push)
pnpm run pre-release      # Pre-release validation
```

### Build Process Details

The build process follows this sequence:

1. **prebuild**: Clean `dist/` and increment build number
2. **test**: Run test suite
3. **build:types**: Generate TypeScript declarations
4. **build**: Bundle with esbuild
5. **postbuild**: Link global installation

**NEVER manually run build steps.** Always use `pnpm run build`.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bobmatnyc/ai-code-review](https://github.com/bobmatnyc/ai-code-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
