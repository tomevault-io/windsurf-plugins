---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Solar Code** (@upstage/solar-cli), a command-line AI workflow tool powered by **Upstage Solar Pro2** that connects to your tools, understands your code, and accelerates your workflows. It's based on the Gemini CLI architecture but enhanced to work with Upstage's Solar Pro2 model, with special optimization for Korean developers and organizations.

## Development Commands

### Building the Project

- `npm run build` - Build all packages
- `npm run build:all` - Build main CLI + sandbox + VS Code companion
- `npm run build:packages` - Build workspace packages only
- `npm run bundle` - Create distribution bundle (includes git commit info generation)

### Development & Testing

- `npm start` - Start CLI in development mode
- `npm run debug` - Start with Node.js debugger attached
- `npm test` - Run all tests across workspaces
- `npm run test:ci` - Run CI tests including script tests
- `npm run test:e2e` - Run end-to-end integration tests
- `npm run test:integration:all` - Run all integration tests (none/docker/podman sandbox variants)

### Code Quality

- `npm run lint` - Lint TypeScript files
- `npm run lint:fix` - Auto-fix linting issues
- `npm run typecheck` - Run TypeScript type checking
- `npm run format` - Format code with Prettier

### Comprehensive Workflow

- `npm run preflight` - Complete quality check pipeline (clean → install → format → lint → build → typecheck → test)

## Architecture Overview

Solar Code follows a **modular monorepo architecture** with distinct separation of concerns, adapted from the Gemini CLI architecture:

### Core Packages Structure

```
packages/
├── cli/           # Frontend: User interface, input handling, display rendering
├── core/          # Backend: Solar API client, tool orchestration, prompt management
├── test-utils/    # Shared testing utilities
└── vscode-ide-companion/  # VS Code extension integration
```

### Key Architectural Patterns

#### 1. **CLI-Core Separation**

- **CLI Package (`packages/cli`)**: Handles user interaction, UI rendering (React/Ink), settings management, Solar authentication flows
- **Core Package (`packages/core`)**: Manages Solar API communication, tool execution, prompt construction, and state management

#### 2. **Solar API Integration**

Located in `packages/core/src/core/`, the Solar integration includes:

- **SolarContentGenerator**: Primary interface to Upstage Solar Pro2 API
- **UpstageConfig**: Configuration validation and management for Solar API keys
- **Solar Types**: TypeScript definitions for Solar API requests and responses
- **JSON Schema Handling**: Converts Gemini's JSON schema requests to Solar-compatible prompts

#### 3. **Authentication System**

Solar Code supports multiple authentication methods:

- **Solar API Key**: Primary method using `UPSTAGE_API_KEY` environment variable
- **API Key Format**: Validates `up_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` format
- **Fallback Support**: Maintains compatibility with other authentication methods for development

#### 4. **Tool System Architecture**

Located in `packages/core/src/tools/`, tools extend Solar's capabilities:

- **File Operations**: read-file, write-file, edit, ls, grep, glob
- **System Integration**: shell, web-fetch, web-search
- **Development**: memory management, MCP (Model Context Protocol) client
- **Tool Lifecycle**: Registration → Validation → User Confirmation → Execution → Result Processing

#### 5. **Request Flow**

1. User input (CLI) → Core package → Solar Pro2 API
2. Solar response → Tool execution (with user approval for destructive operations)
3. Tool results → Solar API → Formatted response → CLI display

#### 6. **Configuration System**

Multi-layered configuration with workspace/user/global settings:

- `packages/cli/src/config/` - Authentication, settings schemas, key bindings
- `packages/core/src/config/upstageConfig.ts` - Solar-specific configuration validation
- `.env.example` - Environment variable template for Solar setup
- Settings merge hierarchy with validation and error handling

## Key Development Concepts

### Solar Pro2 Integration

- **Model Configuration**: Uses `solar-pro2` as the default model
- **API Endpoint**: `https://api.upstage.ai/v1/solar/chat/completions`
- **OpenAI Compatibility**: Solar API follows OpenAI-compatible format with adaptations
- **JSON Schema Handling**: Converts Gemini's responseSchema to prompt instructions for Solar
- **Error Handling**: Provides user-friendly messages for credit/billing issues

### Environment Configuration

Required environment variables for Solar Code:

```bash
UPSTAGE_API_KEY="up_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"  # Required
UPSTAGE_MODEL="solar-pro2"                              # Optional (default)
UPSTAGE_BASE_URL="https://api.upstage.ai/v1/solar"     # Optional (default)
UPSTAGE_MAX_TOKENS=4096                                 # Optional (default)
UPSTAGE_TIMEOUT=120000                                  # Optional (default: 120s)
UPSTAGE_RETRY_COUNT=3                                   # Optional (default: 3)
```

### Memory Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serithemage/solar-code](https://github.com/serithemage/solar-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
