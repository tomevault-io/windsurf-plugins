---
trigger: always_on
description: This is a command-line interface (CLI) tool for managing and interacting with Investec's Programmable Banking API. It allows users to deploy JavaScript code snippets to their programmable bank cards, which execute when transactions are made. The tool also provides commands for managing accounts, balances, transactions, beneficiaries, and more.
---

# Investec Programmable Banking CLI

## Project Overview

This is a command-line interface (CLI) tool for managing and interacting with Investec's Programmable Banking API. It allows users to deploy JavaScript code snippets to their programmable bank cards, which execute when transactions are made. The tool also provides commands for managing accounts, balances, transactions, beneficiaries, and more.

## Technology Stack

- **Runtime**: Node.js 24+ (ESM modules)
- **Language**: TypeScript 5.9+ with strict mode enabled
- **CLI Framework**: Commander.js 14.x
- **Testing**: Vitest 3.x
- **Linting/Formatting**: Biome 2.3.x
- **Build**: TypeScript compiler (tsc)
- **Package Manager**: npm

## Project Structure

```text
src/
  ├── index.ts          # Main entry point, command definitions, global options
  ├── utils.ts          # Shared utilities (error handling, credentials, formatting, etc.)
  ├── errors.ts         # Custom error classes and error codes
  ├── cmds/             # Command implementations
  │   ├── index.ts      # Command exports
  │   ├── types.ts      # Shared TypeScript interfaces
  │   └── *.ts          # Individual command files
  ├── mock-pb.ts        # Mock API for testing/development
  └── mock-card.ts      # Mock card API for testing

test/
  ├── __mocks__/        # Mock implementations for testing
  └── cmds/             # Command-specific tests

bin/                    # Compiled output (generated, not committed)
templates/              # Project templates for `ipb new`
```

## Key Architectural Patterns

### Error Handling

- **Custom Error Class**: `CliError` extends `Error` with error codes
- **Centralized Handler**: `handleCliError()` in `utils.ts` provides consistent error formatting
- **Exit Codes**: `ExitCode` enum for different error types (validation, auth, file, API, etc.)
- **Error Context**: `withCommandContext()` wrapper attaches command names to errors
- **Actionable Messages**: Errors include suggestions for fixing common issues

```typescript
// Example error usage
throw new CliError(ERROR_CODES.MISSING_CARD_KEY, 'Card key is required');
```

### Command Structure

Commands follow a consistent pattern:

1. Validate inputs using utility functions
2. Initialize API clients (with retry/rate limit handling)
3. Show progress indicators (spinners) when appropriate
4. Format output (table, JSON, YAML) based on options
5. Handle errors with context

```typescript
// Example command structure
export async function accountsCommand(options: CommonOptions) {
  const api = await initializePbApi(credentials, options);
  const verbose = getVerboseMode(options.verbose);
  const result = await withRetry(() => api.getAccounts(), { verbose });
  formatOutput(result.data, options);
}
```

### Utility Functions

Key utilities in `src/utils.ts`:

- **Credential Management**: Secure file operations with atomic writes
- **Profile Management**: Multiple credential profiles support
- **Terminal Capabilities**: Unicode/emoji detection and ASCII fallbacks
- **Output Formatting**: Table, JSON, YAML with automatic formatting
- **Rate Limiting**: Automatic retry with exponential backoff
- **Progress Indicators**: File size-aware spinners
- **Secret Detection**: Warnings for insecure secret usage
- **Command History**: Logging of executed commands

### Configuration Management

- **Credentials**: Stored in `~/.ipb/.credentials.json` with restricted permissions (600)
- **Profiles**: Multiple profiles in `~/.ipb/profiles/<name>.json`
- **Active Profile**: Tracked in `~/.ipb/active-profile.json`
- **Priority Order**: Command args → Profile → Environment vars → Credentials file
- **Security**: Warns when secrets are loaded from environment variables

### Testing

- **Framework**: Vitest with ESM support
- **Mocking**: `vi.mock()` for modules, `vi.hoisted()` for shared mocks
- **Test Structure**: Commands tested in `test/cmds/*.test.ts`
- **Mocks**: Located in `test/__mocks__/` for external dependencies
- **Coverage**: Configured in `vitest.config.ts`

## Coding Standards

### TypeScript

- **Strict Mode**: Enabled with `noUncheckedIndexedAccess` and `noImplicitOverride`
- **Module System**: ESM only (`"type": "module"`)
- **Target**: ES2022
- **Imports**: Use `.js` extension for ESM imports (TypeScript requirement)

### Linting & Formatting

- **Tool**: Biome (replaces ESLint/Prettier)
- **Configuration**: `biome.json`
- **Rules**:
  - `noExplicitAny`: Error (use `biome-ignore` comments when necessary)
  - `noUnusedVariables`: Error
  - Recommended rules enabled
- **Auto-fix**: Run `npm run lint:fix` to auto-fix issues

### Code Style

- **Quotes**: Single quotes
- **Semicolons**: Always
- **Indentation**: 2 spaces
- **Line Width**: 100 characters
- **Trailing Commas**: ES5 style
- **Arrow Functions**: Always parentheses

### Import Organization

- Biome automatically organizes imports when running `lint:fix`
- Grouping: External packages → Node.js built-ins → Internal/relative → Type imports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devinpearson/ipb](https://github.com/devinpearson/ipb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
