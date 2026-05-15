---
trigger: always_on
description: This file provides guidance to agentic coding agents like [Claude Code](https://claude.ai/code), [Gemini CLI](https://github.com/google-gemini/gemini-cli), and [Codex CLI](https://github.com/openai/codex) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to agentic coding agents like [Claude Code](https://claude.ai/code), [Gemini CLI](https://github.com/google-gemini/gemini-cli), and [Codex CLI](https://github.com/openai/codex) when working with code in this repository.

## Project Overview

llmc is an AI-powered commit message generator that creates Conventional Commits-compliant messages from git diffs. It's a Node.js/TypeScript CLI tool that supports multiple AI providers (Anthropic, OpenAI, Google, etc.) and can be integrated with git hooks. The tool offers two modes: automatic commit mode (default) and message-only mode for git hook integration.

## Architecture

### Core Components

- **index.ts**: Main CLI entry point and orchestrator with command-line argument parsing for message-only mode
- **app.tsx**: React-based application logic with git integration, CLI rendering, and dual-mode support
- **message.ts**: Core message generation logic with AI provider integration- **providers.ts**: Dynamic provider loading system supporting 13+ AI services
- **config.ts**: Configuration management with TOML support and type-safe defaults
- **cli.tsx**: React component for CLI interface with ink for progress indicators and status messages

### Key Design Patterns

- **Dynamic Provider Loading**: Providers are loaded on-demand to reduce bundle size
- **Configuration Hierarchy**: TOML config file → environment variables → sensible defaults
- **Structured Generation**: Uses `generateObject` with Zod schema for reliable output
- **Template Interpolation**: Custom prompts support `${diff}` placeholders
- **React-based CLI**: Uses ink for rich terminal UI with spinners and real-time status updates
- **Separation of Concerns**: index.ts remains simple .ts for compatibility, React/JSX logic in .tsx files
- **Snake/Camel Case Conversion**: Automatic conversion between TOML snake_case and TypeScript camelCase
- **Comprehensive Testing**: Full test coverage with unit, integration, and UI testing strategies

## Testing

- Uses Vitest as the test runner with `ink-testing-library` for React component testing
- API integration tests are separated into their own file (`integration-api.test.ts`) and require explicit execution
- Regular integration tests run build verification and basic CLI functionality without API calls
- Tests cover CLI argument handling, config loading, build verification, and React UI components
- Comprehensive test coverage achieved: ~90% overall, 100% for core modules
- Timer increment testing works well with Vitest's timer mocking capabilities

### Test Files

- `cli.test.tsx`: React component rendering and status transitions
- `app.test.tsx`: Application logic, React UI testing, and module exports
- `index.test.ts`: Entry point module structure
- `integration.test.ts`: Built executable and basic CLI functionality (no API calls)
- `integration-api.test.ts`: API integration tests requiring valid API key (run separately)
- `message.test.ts`: Core message generation functionality with 100% coverage
- `config.test.ts`: Configuration management and type system validation
- `providers.test.ts`: Provider loading and factory functions
- `message-only.test.ts`: Message-only mode functionality and argument parsing

### Test Coverage Strategy

- **Unit Tests**: Individual functions and components with comprehensive mocking
- **Integration Tests**: Real CLI execution and git integration (separated by API usage)
- **API Tests**: Separate test file for full end-to-end API integration (`npm run test:integration`)
- **UI Tests**: React component behavior using `ink-testing-library`
- **Error Path Testing**: All failure scenarios and edge cases covered
- **Mock Strategy**: Extensive use of Vitest's `vi.mock()` for dependency isolation

## Configuration

The application uses a `llmc.toml` file in the project root with these key settings:

- `provider`: AI service to use (defaults to "anthropic")
- `model`: Specific model name
- `max_tokens`: Response length limit (snake_case in TOML)
- `api_key`: Explicit API key (snake_case in TOML)
- `api_key_name`: Environment variable name for API key (snake_case in TOML)
- `temperature`: AI creativity level
- `prompt`: Custom message template with `${diff}` interpolation

### Configuration File Format

The configuration uses **snake_case** naming in TOML files but is automatically converted to **camelCase** in TypeScript runtime:

```toml
# llmc.toml (snake_case)
provider = "openai"
max_tokens = 200
api_key = "your-key"
api_key_name = "OPENAI_API_KEY"
```

```typescript
// Runtime config (camelCase)
{
  provider: "openai",
  maxTokens: 200,      // Converted from max_tokens
  apiKey: "your-key",  // Converted from api_key
  apiKeyName: "OPENAI_API_KEY"  // Converted from api_key_name
}
```

### Configuration Loading

- **Automatic Conversion**: Built-in `toCamelCase()` utility converts snake_case TOML keys to camelCase TypeScript properties
- **Graceful Fallbacks**: Missing config file or parsing errors fall back to sensible defaults

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marclove/llmc](https://github.com/marclove/llmc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
