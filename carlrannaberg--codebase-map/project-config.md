---
trigger: always_on
description: This file provides guidance to AI coding assistants working in this repository.
---

# AGENT.md
This file provides guidance to AI coding assistants working in this repository.

**Note:** CLAUDE.md, .clinerules, .cursorrules, .windsurfrules, and other AI config files are symlinks to AGENT.md in this project.

# Code Map - TypeScript Code Indexer

A lightweight, self-contained TypeScript/JavaScript code indexing tool that generates comprehensive maps of project structure, dependencies, and code signatures. This tool provides fast analysis of codebases by extracting file trees, inter-file dependencies, and essential code metadata without requiring a TypeScript compiler or complex toolchain.

## Build & Commands

### Initial Setup (REQUIRED FIRST)
**CRITICAL**: This project is not yet initialized. You must first set up the Node.js project:

```bash
# Initialize Node.js project with ESM support
npm init -y

# Update package.json to include type: "module" and required scripts
# Install core dependencies
npm install fast-glob@^3 ignore@^7 tsx@^4 typescript@^5

# Install development dependencies
npm install --save-dev @types/node vitest @vitest/coverage-v8 eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser prettier
```

### Development Commands (After Setup)
**Note**: These commands will be available after initializing the project:

- **Development**: `npm run dev` (run with tsx)
- **Build**: `npm run build` (compile TypeScript)
- **Test**: `npm test` (run Vitest tests)
- **Test with coverage**: `npm run test:coverage`
- **Type check**: `npm run typecheck` (tsc --noEmit)
- **Lint**: `npm run lint` (ESLint)
- **Format**: `npm run format` (Prettier)
- **Clean**: `npm run clean` (remove build artifacts)

### CLI Usage (Planned)
```bash
# Full project scan
npx tsx src/indexer.ts scan [root]

# Update single file in index
npx tsx src/indexer.ts update <file>
```

### Script Command Consistency
**Important**: When modifying npm scripts in package.json, ensure all references are updated:
- GitHub Actions workflows (.github/workflows/*.yml)
- README.md documentation
- Contributing guides
- Claude Code hooks in .claude/settings.json

## Code Style

### Language & Framework
- **Language**: TypeScript 5.x with strict mode
- **Module System**: ESM (ECMAScript Modules) - use `import`/`export`
- **Runtime**: Node.js 18+ with native ESM support
- **Execution**: tsx for development (no compilation needed)

### TypeScript Configuration
```typescript
// Recommended tsconfig.json settings
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "allowJs": false,
    "noEmit": true,
    "types": ["node", "vitest/globals"]
  }
}
```

### Import Conventions
- Use ESM imports exclusively (no `require()`)
- Prefer named exports over default exports
- Group imports: external deps, then internal modules
- Use `.js` extensions for relative imports in TypeScript files
```typescript
// External dependencies
import { glob } from 'fast-glob';
import ignore from 'ignore';

// Internal modules (note .js extension even for .ts files)
import { parseFile } from './parser.js';
import type { FileNode, ProjectIndex } from './types.js';
```

### Naming Conventions
- **Files**: kebab-case (`file-discovery.ts`, `ast-parser.ts`)
- **Classes**: PascalCase (`FileDiscovery`, `AstParser`)
- **Interfaces/Types**: PascalCase with descriptive names
- **Functions**: camelCase, verb-prefixed (`parseFile`, `buildTree`)
- **Constants**: UPPER_SNAKE_CASE for true constants
- **Private methods**: prefix with underscore (`_processNode`)

### Error Handling
- Use structured error types for different failure modes
- Provide context in error messages (file path, line number)
- Never swallow errors silently
- Log warnings for recoverable issues
```typescript
class ParseError extends Error {
  constructor(
    message: string,
    public readonly filePath: string,
    public readonly line?: number
  ) {
    super(`${filePath}${line ? `:${line}` : ''}: ${message}`);
    this.name = 'ParseError';
  }
}
```

### Type Usage Patterns
- Prefer interfaces for object shapes
- Use type aliases for unions, intersections, and complex types
- Always specify return types explicitly
- Use `readonly` for immutable properties
- Leverage discriminated unions for state
```typescript
interface FileNode {
  readonly path: string;
  readonly type: 'file' | 'directory';
  dependencies?: string[];
}

type ParseResult = 
  | { success: true; data: FileNode }
  | { success: false; error: Error };
```

## Testing

### Testing Framework
- **Framework**: Vitest (Jest-compatible API)
- **Test files**: `*.test.ts` or `*.spec.ts` alongside source files
- **Coverage**: Vitest with v8 coverage provider
- **Assertions**: Use Vitest's `expect` API

### Testing Conventions
```typescript
// File: src/parser.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { parseFile } from './parser.js';

describe('parseFile', () => {
  it('should extract function signatures', async () => {
    const result = await parseFile('test.ts');
    expect(result.functions).toHaveLength(2);
  });
});
```

### Running Tests
```bash
# Run all tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlrannaberg/codebase-map](https://github.com/carlrannaberg/codebase-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
