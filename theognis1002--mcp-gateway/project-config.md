---
trigger: always_on
description: This guide provides comprehensive instructions for AI coding agents on working with the MCP Gateway Gateway frontend test suite. The project uses Jest with jsdom environment and follows a structured testing approach.
---

# Frontend Testing Guide - MCP Gateway Gateway

## Overview

This guide provides comprehensive instructions for AI coding agents on working with the MCP Gateway Gateway frontend test suite. The project uses Jest with jsdom environment and follows a structured testing approach.

## Testing Architecture

### Framework & Tools
- **Test Runner**: Jest with jsdom environment 
- **React Testing**: React Testing Library (for component tests)
- **API Mocking**: Mock Service Worker (MSW) for integration tests
- **File Naming**: `*.simple.test.ts` for unit tests, `*.test.ts` for integration tests
- **Configuration**: Uses `jest.simple.config.cjs` for simple unit tests

### Test Categories

#### 1. Simple Unit Tests (`*.simple.test.ts`)
- **Purpose**: Fast, isolated tests with minimal dependencies
- **Mocking**: Uses Jest `mockFn()` to mock `fetch` globally
- **Environment**: jsdom with polyfills for browser APIs
- **Focus**: Pure functions, API clients, validation, utilities

#### 2. Integration Tests (`*.test.ts`) 
- **Purpose**: Tests with MSW server, React components
- **Mocking**: Uses Mock Service Worker for realistic API interactions
- **Environment**: Full browser simulation with MSW handlers
- **Focus**: Component behavior, user interactions, API integration

## Project Structure

### Test Directory Structure
```
src/test/
├── __mocks__/          # Mock files for modules
├── handlers/           # MSW request handlers
├── polyfills.ts        # Browser API polyfills
├── server.ts           # MSW server setup
├── setup-simple.ts     # Jest setup for simple tests
└── setup.ts           # Jest setup for integration tests
```

### Test File Locations
```
src/lib/__tests__/                    # API client tests
src/lib/validation/__tests__/         # Schema validation tests  
src/hooks/__tests__/                  # Custom hook tests
src/@auth/__tests__/                  # Auth context tests
src/app/(controlpanel)/*/api/hooks/__tests__/  # TanStack Query hooks
```

## Test Types & Coverage Areas

### API Client Testing
- **Authentication API** (`src/lib/auth-api.ts`)
  - File: `src/lib/__tests__/auth-api-extended.simple.test.ts`
  - Tests: Token management, auth flow, API key CRUD, error handling

- **Client API** (`src/lib/client-api.ts`)
  - File: `src/lib/__tests__/client-api.simple.test.ts`
  - Tests: All API classes, CRUD operations, HTTP status handling

- **Configuration API** (`src/lib/config-api.ts`)
  - File: `src/lib/__tests__/config-api.simple.test.ts` 
  - Tests: Export/import functionality, FormData handling, Blob responses

### Validation & Data Integrity
- **Validation Schemas** (`src/lib/validation/`)
  - Files: `src/lib/validation/__tests__/*.simple.test.ts`
  - Tests: Zod schema validation, constraint validation, enum validation

- **Utility Functions** (`src/lib/utils.ts`)
  - File: `src/lib/__tests__/utils.simple.test.ts`
  - Tests: `cn` function, className merging, Tailwind conflicts

### React Hooks & State Management
- **Custom Hooks** (`src/hooks/`)
  - Files: `src/hooks/__tests__/*.simple.test.ts`
  - Tests: Hook behavior, state updates, cleanup functions, edge cases

- **Authentication Context** (`src/@auth/AuthContext.tsx`)
  - File: `src/@auth/__tests__/AuthContext.simple.test.tsx`
  - Tests: Provider initialization, auth state, token refresh

### Data Fetching & API Integration
- **TanStack Query Hooks**
  - Files: `src/app/(controlpanel)/*/api/hooks/__tests__/*.simple.test.ts`
  - Tests: Query behavior, mutations, cache management, error states

## Test Implementation Guidelines

### File Structure Pattern
```typescript
import { apiClient } from '../api-module';

// Mock fetch globally for simple tests
const mockFetch = jest.fn();
global.fetch = mockFetch;

describe('API Module - Simple Tests', () => {
  beforeEach(() => {
    mockFetch.mockClear();
    // Clear storage if needed
    if (typeof localStorage !== 'undefined') {
      localStorage.clear();
    }
  });

  describe('method group', () => {
    it('should do expected behavior', async () => {
      // Arrange: Setup mocks
      mockFetch.mockResolvedValueOnce({
        ok: true,
        status: 200,
        headers: { get: jest.fn().mockReturnValue('application/json') },
        json: jest.fn().mockResolvedValueOnce({ data: 'expected' })
      });

      // Act: Call the function
      const result = await apiClient.someMethod();

      // Assert: Verify behavior
      expect(mockFetch).toHaveBeenCalledWith(/* expected args */);
      expect(result).toEqual(/* expected result */);
    });
  });
});
```

### Mock Patterns

#### 1. Fetch API Mocking (Simple Tests)
```typescript
// Success response
mockFetch.mockResolvedValueOnce({
  ok: true,
  status: 200,
  headers: { get: jest.fn().mockReturnValue('application/json') },
  json: jest.fn().mockResolvedValueOnce({ data: 'result' })
});

// Error response  
mockFetch.mockResolvedValueOnce({
  ok: false,
  status: 400,
  statusText: 'Bad Request'
});

// Network error
mockFetch.mockRejectedValueOnce(new Error('Network error'));
```

#### 2. LocalStorage Handling
```typescript
beforeEach(() => {
  // Clear storage safely

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theognis1002/mcp-gateway](https://github.com/theognis1002/mcp-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
