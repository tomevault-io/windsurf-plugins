---
trigger: always_on
description: TypeScript coding standards, naming conventions, code organization, and best practices
---


# Coding Standards

## TypeScript Standards

### Type Safety

```typescript
// ✅ GOOD: Explicit types
export function generateHiddenText(text: string, style: HiddenTextStyle): string {
  return transformText(text, style);
}

// ❌ BAD: Any types
export function generateHiddenText(text: any, style: any): any {
  return transformText(text, style);
}
```

### Null Safety

```typescript
// ✅ GOOD: Optional chaining
const fileName = editor?.document?.fileName;

// ❌ BAD: Unsafe access
const fileName = editor.document.fileName;

// ✅ GOOD: Nullish coalescing
const timeout = config.get('timeout') ?? 300;

// ❌ BAD: Falsy check
const timeout = config.get('timeout') || 300; // 0 would be replaced!
```

### Function Declarations

```typescript
// ✅ GOOD: Pure function with clear contract
export function matchPattern(key: string, patterns: string[]): boolean {
  if (patterns.length === 0) return false;
  return patterns.some((pattern) => new RegExp(pattern, 'i').test(key));
}

// ❌ BAD: Side effects without documentation
export function matchPattern(key: string, patterns: string[]): boolean {
  cache.set(key, patterns); // Hidden side effect!
  return patterns.some((pattern) => new RegExp(pattern, 'i').test(key));
}
```

### Async/Await

```typescript
// ✅ GOOD: Error handling
async function saveConfig(data: ConfigData): Promise<void> {
  try {
    await fs.writeFile(CONFIG_PATH, JSON.stringify(data));
  } catch (error) {
    console.error('Failed to save config:', error);
    throw new Error('Config save failed');
  }
}

// ❌ BAD: Unhandled promise
function saveConfig(data: ConfigData): void {
  fs.writeFile(CONFIG_PATH, JSON.stringify(data)); // Promise ignored!
}
```

## Naming Conventions

### Variables and Functions

- `camelCase` for variables and functions
- Descriptive names over short names
- Boolean variables start with `is`, `has`, `should`, `can`

```typescript
// ✅ GOOD
const isEnabled = true;
const hasPatterns = patterns.length > 0;
function shouldHideValue(key: string): boolean {}

// ❌ BAD
const enabled = true; // Ambiguous
const patterns_exist = patterns.length > 0; // Wrong case
function hideValue(key: string): boolean {} // Unclear return type expectation
```

### Classes and Interfaces

- `PascalCase` for classes and interfaces
- Interfaces describe behavior or shape
- Avoid `I` prefix for interfaces

```typescript
// ✅ GOOD
export class Camouflage {}
export interface TextDocumentContentProvider {}
export enum HiddenTextStyle {}

// ❌ BAD
export class camouflage {}
export interface textProvider {}
export enum hiddenStyle {}
```

### Constants

- `UPPER_SNAKE_CASE` for true constants
- `camelCase` for configuration values

```typescript
// ✅ GOOD
const MAX_CACHE_SIZE = 100;
const DEFAULT_TIMEOUT = 300;
const envFilePatterns = ['.env', '.env.local'];

// ❌ BAD
const max_cache_size = 100; // Wrong case
const ENVFILEPATTERNS = ['.env']; // Config, not constant
```

## Code Organization

### Imports

```typescript
// ✅ GOOD: Grouped and organized
import * as fs from 'fs';
import * as path from 'path';

import * as vscode from 'vscode';

import { generateHiddenText } from '../lib/text-generator';
import { isEnvFile } from '../utils/file-utils';
import * as config from '../utils/config';

import type { HiddenTextStyle } from '../types';
```

**Order**:

1. Node.js built-ins
2. External packages
3. Internal modules (grouped by directory)
4. Type-only imports

### File Structure

```typescript
// 1. Imports
import * as vscode from 'vscode';

// 2. Types and Interfaces
export interface CamouflageConfig {
  enabled: boolean;
}

// 3. Constants
const DEFAULT_STYLE = 'text';

// 4. Main class/functions
export class Camouflage {
  // Public properties first
  public readonly version: string;

  // Private properties after
  private decorationType?: vscode.TextEditorDecorationType;

  // Constructor
  constructor() {}

  // Public methods
  public initialize(): void {}

  // Private methods
  private updateDecorations(): void {}
}

// 5. Helper functions (if needed)
function helperFunction(): void {}
```

## Comments and Documentation

### JSDoc

````typescript
/**
 * Masks sensitive values in environment files
 *
 * @param content - The file content to mask
 * @param style - The hiding style to apply
 * @returns Masked content with hidden values
 *
 * @example
 * ```typescript
 * const masked = maskContent('API_KEY=secret', 'stars');
 * // Returns: 'API_KEY=******'
 * ```
 */
export function maskContent(content: string, style: HiddenTextStyle): string {
  // ...
}
````

### Inline Comments

```typescript
// ✅ GOOD: Explain WHY, not WHAT
// Debounce to prevent excessive decoration updates during rapid typing
@Debounce(100)
private updateDecorations(): void { }

// ❌ BAD: Explain WHAT (code already shows this)
// This function updates decorations
private updateDecorations(): void { }
```

### TODO Comments

```typescript
// ✅ GOOD: Include context and issue number
// TODO(#45): Add support for .env.production files

// ❌ BAD: Vague TODO
// TODO: Fix this
```

## Performance Best Practices

### Debouncing

```typescript
// ✅ GOOD: Debounce frequent operations
@Debounce(100)
private updateDecorations(): void {
  // Called max once per 100ms
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeybek/camouflage](https://github.com/zeybek/camouflage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
