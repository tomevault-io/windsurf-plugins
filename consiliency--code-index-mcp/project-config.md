---
trigger: always_on
description: This file defines TypeScript/JavaScript coding standards and patterns for any web UI or Node.js components that may be added to the Code-Index-MCP project.
---

# TypeScript Rules for Code-Index-MCP

## Overview
This file defines TypeScript/JavaScript coding standards and patterns for any web UI or Node.js components that may be added to the Code-Index-MCP project.

## Type Safety

### Strict Type Checking
```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### Type Definitions
```typescript
// Define interfaces for API responses
interface SymbolDefinition {
  name: string;
  type: 'function' | 'class' | 'variable' | 'method';
  filePath: string;
  line: number;
  column: number;
  docstring?: string;
}

interface SearchResult {
  matches: Array<{
    file: string;
    line: number;
    content: string;
    score: number;
  }>;
  totalCount: number;
}

// Use discriminated unions for different response types
type APIResponse<T> = 
  | { status: 'success'; data: T }
  | { status: 'error'; error: string }
  | { status: 'loading' };
```

## API Client Design

### Type-Safe API Client
```typescript
class MCPClient {
  private baseUrl: string;
  private apiKey: string;

  constructor(baseUrl: string, apiKey: string) {
    this.baseUrl = baseUrl;
    this.apiKey = apiKey;
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': this.apiKey,
        ...options.headers,
      },
    });

    if (!response.ok) {
      throw new Error(`API Error: ${response.statusText}`);
    }

    return response.json();
  }

  async getSymbolDefinition(
    symbolName: string,
    filePath?: string
  ): Promise<SymbolDefinition> {
    return this.request<SymbolDefinition>('/symbol', {
      method: 'POST',
      body: JSON.stringify({ symbol_name: symbolName, file_path: filePath }),
    });
  }

  async searchCode(
    query: string,
    fileExtensions?: string[]
  ): Promise<SearchResult> {
    return this.request<SearchResult>('/search', {
      method: 'POST',
      body: JSON.stringify({ query, file_extensions: fileExtensions }),
    });
  }
}
```

## Error Handling

### Custom Error Classes
```typescript
class MCPError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode?: number
  ) {
    super(message);
    this.name = 'MCPError';
  }
}

class NetworkError extends MCPError {
  constructor(message: string) {
    super(message, 'NETWORK_ERROR');
  }
}

class AuthenticationError extends MCPError {
  constructor(message: string) {
    super(message, 'AUTH_ERROR', 401);
  }
}
```

### Error Handling Patterns
```typescript
// Result type for safer error handling
type Result<T, E = Error> = 
  | { ok: true; value: T }
  | { ok: false; error: E };

async function safeApiCall<T>(
  fn: () => Promise<T>
): Promise<Result<T>> {
  try {
    const value = await fn();
    return { ok: true, value };
  } catch (error) {
    return { 
      ok: false, 
      error: error instanceof Error ? error : new Error(String(error))
    };
  }
}

// Usage
const result = await safeApiCall(() => 
  client.getSymbolDefinition('myFunction')
);

if (result.ok) {
  console.log('Symbol found:', result.value);
} else {
  console.error('Error:', result.error.message);
}
```

## State Management

### React State Pattern (if UI is added)
```typescript
import { create } from 'zustand';

interface CodeIndexState {
  symbols: Map<string, SymbolDefinition>;
  searchResults: SearchResult | null;
  isLoading: boolean;
  error: string | null;
  
  // Actions
  searchCode: (query: string) => Promise<void>;
  getSymbol: (name: string) => Promise<void>;
  clearError: () => void;
}

const useCodeIndexStore = create<CodeIndexState>((set, get) => ({
  symbols: new Map(),
  searchResults: null,
  isLoading: false,
  error: null,

  searchCode: async (query: string) => {
    set({ isLoading: true, error: null });
    try {
      const results = await client.searchCode(query);
      set({ searchResults: results, isLoading: false });
    } catch (error) {
      set({ 
        error: error instanceof Error ? error.message : 'Unknown error',
        isLoading: false 
      });
    }
  },

  getSymbol: async (name: string) => {
    const { symbols } = get();
    if (symbols.has(name)) return;

    set({ isLoading: true, error: null });
    try {
      const symbol = await client.getSymbolDefinition(name);
      set((state) => ({
        symbols: new Map(state.symbols).set(name, symbol),
        isLoading: false
      }));
    } catch (error) {
      set({ 
        error: error instanceof Error ? error.message : 'Unknown error',
        isLoading: false 
      });
    }
  },

  clearError: () => set({ error: null })
}));
```

## Testing Patterns

### Unit Testing
```typescript
import { describe, it, expect, vi } from 'vitest';

describe('MCPClient', () => {
  it('should fetch symbol definition', async () => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Consiliency/Code-Index-MCP](https://github.com/Consiliency/Code-Index-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
