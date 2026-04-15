---
trigger: always_on
description: - **Node.js**: >=24.0.0 (ESM modules only)
---

# Investec Programmable Banking CLI - Cursor Rules

## Project Technology Stack

### Core Technologies
- **Node.js**: >=24.0.0 (ESM modules only)
- **TypeScript**: 5.9.3 with strict mode
- **Module System**: ESM (ES Modules) - all imports use `.js` extension
- **CLI Framework**: Commander.js 14.x
- **Testing**: Vitest 3.x with ESM support
- **Linting/Formatting**: Biome 2.3.x (replaces ESLint/Prettier)
- **Build Tool**: TypeScript compiler (tsc)

### Key Dependencies
- `commander`: CLI framework for command definitions
- `chalk`: Terminal colors (v5, respects NO_COLOR/FORCE_COLOR)
- `ora`: Spinner/progress indicators
- `cli-table3`: Enhanced table formatting
- `js-yaml`: YAML serialization
- `@inquirer/prompts`: Interactive prompts (confirm, input, password)
- `investec-pb-api`: Programmable Banking API client
- `investec-card-api`: Card API client
- `programmable-card-code-emulator`: Local code execution emulator

## Project Setup

### Directory Structure
```
src/
  ├── index.ts          # Entry point, command registration, global options
  ├── utils.ts          # Shared utilities (2500+ lines of helper functions)
  ├── errors.ts         # CliError class, ExitCode enum, ERROR_CODES
  ├── cmds/             # Command implementations (30+ commands)
  │   ├── index.ts      # Command exports
  │   ├── types.ts      # Shared interfaces (CommonOptions, Credentials, etc.)
  │   └── *.ts          # Individual command files
  ├── mock-pb.ts        # Mock Programmable Banking API for testing
  └── mock-card.ts      # Mock Card API for testing

test/
  ├── __mocks__/        # Mock implementations (ora, utils, external-editor, etc.)
  ├── cmds/             # Command tests
  ├── helpers.ts        # Test helper utilities
  └── setup.js          # Vitest setup

bin/                    # Compiled output (gitignored, generated)
templates/              # Project templates (default, petro)
```

### Build & Development
- **Build**: `npm run build` → TypeScript compiles `src/` to `bin/`
- **Source Maps**: Enabled for debugging
- **Declaration Files**: Generated for library consumers
- **File Copying**: Templates and assets copied to `bin/` during build

### TypeScript Configuration
- **Target**: ES2022
- **Module**: NodeNext (ESM)
- **Strict Mode**: Enabled with `noUncheckedIndexedAccess` and `noImplicitOverride`
- **Module Resolution**: NodeNext
- **Import Extensions**: Must use `.js` for ESM imports (TypeScript requirement)

## Code Patterns & Conventions

### Error Handling Pattern
```typescript
// 1. Use CliError for user-facing errors
throw new CliError(ERROR_CODES.MISSING_CARD_KEY, 'Card key is required');

// 2. Wrap commands with context
export const myCommand = withCommandContext('my-command', async (options) => {
  // Command implementation
});

// 3. Centralized error handling in main()
catch (error) {
  handleCliError(error, { verbose: getVerboseMode(true) }, 'command name');
}
```

### Command Structure Pattern
```typescript
export async function commandName(options: CommonOptions) {
  // 1. Validate inputs
  const normalizedPath = await validateFilePath(options.filename, ['.js']);
  
  // 2. Initialize API with retry support
  const api = await initializePbApi(credentials, options);
  const verbose = getVerboseMode(options.verbose);
  
  // 3. Show progress (if not piped)
  const spinner = createSpinner(!isStdoutPiped(), getSafeText('💳 Processing...'));
  
  // 4. Make API calls with retry
  const result = await withRetry(() => api.someMethod(), { verbose });
  
  // 5. Format output
  formatOutput(result.data, options);
}
```

### Utility Functions Usage
- **Credentials**: `loadCredentialsFile()`, `readCredentialsFile()`, `writeCredentialsFile()`
- **Profiles**: `getActiveProfile()`, `setActiveProfile()`, `readProfile()`, `deleteProfile()`
- **Terminal**: `getSafeText()`, `detectTerminalCapabilities()`, `getTerminalDimensions()`
- **Output**: `formatOutput()`, `printTable()`, `formatFileSize()`
- **Validation**: `validateAmount()`, `validateAccountId()`, `validateFilePath()`
- **API**: `initializeApi()`, `initializePbApi()`, `withRetry()`
- **Security**: `warnAboutSecretUsage()`, `detectSecretUsageFromEnv()`
- **History**: `logCommandHistory()`, `readCommandHistory()`

### Import Patterns
```typescript
// Node.js built-ins (use node: prefix)
import { promises as fsPromises } from 'node:fs';
import { homedir } from 'node:os';

// External packages
import chalk from 'chalk';
import { Command } from 'commander';

// Internal modules (use .js extension)
import { CliError, ERROR_CODES } from '../errors.js';
import { formatOutput, getVerboseMode } from '../utils.js';

// Type-only imports
import type { CommonOptions, Credentials } from './types.js';
```

## Testing Patterns

### Mock Setup
```typescript
vi.mock('../../src/utils.ts', async () => {
  const actual = await vi.importActual<typeof import('../../src/utils.ts')>();
  return {
    ...actual,
    initializeApi: vi.fn(),
    createSpinner: vi.fn(() => ({
      start: vi.fn(function() { return this; }),
      stop: vi.fn(),
      text: '',
    })),
  };
});
```

### Test Structure
- Use `describe()` blocks for grouping
- Use `it()` for individual test cases
- Mock external dependencies
- Test both success and error paths
- Use `expect().toThrow(CliError)` for error testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devinpearson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
