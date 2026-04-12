---
trigger: always_on
description: Code searchability patterns and ast-grep integration
---

## Overview

This codebase uses **ast-grep** for structural code search and pattern matching. ast-grep provides semantic search beyond simple text matching, understanding TypeScript/JavaScript syntax.

See [sgconfig.yml](mdc:sgconfig.yml) and [rules/](mdc:rules/) for complete configuration.


## Quick Reference

### Common Search Commands

```bash
# Search for specific patterns
ast-grep --pattern 'async function $NAME' src/
sg -p 'const $VAR = fetch($$$)' src/

# Filter by specific rules
ast-grep scan --filter "sql-injection-risk"
sg scan --filter "unstructured-log"

# Scan all rules in project
ast-grep scan
sg scan

# Search in specific files
ast-grep --pattern 'throw Errors.$METHOD($$$)' src/api/
```

### Search by Feature

| Feature | Command | Location |
|---------|---------|----------|
| **SQL Injection** | `sg scan --filter "sql-injection-risk"` | All TS files |
| **CORS Missing** | `sg scan --filter "missing-cors-headers"` | All TS files |
| **Unstructured Logs** | `sg scan --filter "unstructured-log"` | All TS files |
| **Hardcoded URLs** | `sg scan --filter "hardcoded-url"` | All files |
| **WORKER_URL Dupe** | `sg scan --filter "worker-url-definition"` | Dashboard files |


## Installation

Install ast-grep globally:

```bash
# Using cargo (recommended)
cargo install ast-grep

# Using npm
npm install -g @ast-grep/cli

# Using brew (macOS)
brew install ast-grep
```

Verify installation:

```bash
sg --version
```


## Usage Patterns

### 1. Finding API Endpoints

```bash
# Find all endpoint handlers
ast-grep --pattern 'async function $NAME(request: Request, env: Env, requestId: string)' src/

# Find specific endpoint
sg -p 'async function getEvents' src/api/

# Find endpoints without requestId
sg scan --filter "missing-request-id"
```

**Example Output:**
```
src/api/routes.ts:45:1
async function getEvents(request: Request, env: Env, requestId: string): Promise<Response> {
  // Implementation
}
```

### 2. Finding MCP Handlers

```bash
# Find all MCP tool handlers  
ast-grep --pattern 'export async function $NAME(params: $PARAMS, env: Env): Promise<MCPToolResult>' src/mcp/

# Find specific handler
sg -p 'export async function getSteamMoves' src/

# Find handlers returning MCPToolResult
sg -p 'Promise<MCPToolResult>' src/
```

### 3. Finding Database Queries

```bash
# Find all D1 queries
ast-grep --pattern 'env.$DB.prepare($QUERY)' src/

# Find queries with specific table
sg -p "env.ANALYTICS.prepare(\`SELECT * FROM line_movements\`)" src/

# Find potential SQL injection risks
sg scan --filter "sql-injection-risk"
```

**Anti-Pattern Detection:**
```typescript
// ❌ BAD: String interpolation (detected by ast-grep)
env.ANALYTICS.prepare(`SELECT * FROM table WHERE id = '${id}'`);

// ✅ GOOD: Parameterized query
env.ANALYTICS.prepare(`SELECT * FROM table WHERE id = ?`).bind(id);
```

### 4. Finding Validation Calls

```bash
# Find all validation calls
ast-grep --pattern 'validateQueryParams($URL, $VALIDATORS)' src/

# Find specific validator
sg -p 'Validators.agentID' src/
sg -p 'validateQueryParams' src/
```

### 5. Finding Error Handling

```bash
# Find all error throws
ast-grep --pattern 'throw Errors.$METHOD($$$)' src/

# Find specific error type
sg -p 'throw Errors.validationError' src/
sg -p 'throw Errors.notFound' src/
```

### 6. Finding Console Logs

```bash
# Find all console logs
ast-grep --pattern 'console.$METHOD($$$)' src/

# Find unstructured logs (missing requestId)
sg scan --filter "unstructured-log"
sg scan --filter "unstructured-log" src/index.ts  # Specific file

# Find specific log level
sg -p 'console.error' src/
```


## Code Quality Rules

### Security Rules

#### 1. SQL Injection Prevention
```bash
sg scan --filter "sql-injection-risk"
```

**Rule:** Never use string interpolation in SQL queries.

```typescript
// ❌ WRONG: SQL injection risk
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// ✅ CORRECT: Parameterized query
const query = env.ANALYTICS.prepare(`SELECT * FROM users WHERE id = ?`).bind(userId);
```

#### 2. CORS Headers
```bash
sg scan --filter "missing-cors-headers"
```

**Rule:** All API responses must include CORS headers.

```typescript
// ❌ WRONG: Missing CORS
return new Response(JSON.stringify(data));

// ✅ CORRECT: With CORS
return new Response(JSON.stringify(data), {
  headers: {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*'
  }
});
```

### Observability Rules

#### 1. Request ID Tracking
```bash
sg scan --filter "missing-request-id"
```

**Rule:** All endpoint handlers must generate and use requestId.

```typescript
// ❌ WRONG: Missing requestId
async function getEvents(request: Request, env: Env): Promise<Response> {
  console.log('Fetching events');
  // ...
}

// ✅ CORRECT: With requestId
async function getEvents(request: Request, env: Env, requestId: string): Promise<Response> {
  console.log(`[${requestId}] Fetching events`);
  // ...
}
```

#### 2. Structured Logging
```bash
sg scan --filter "unstructured-log"
```

**Rule:** All logs should include requestId prefix.

```typescript
// ❌ WRONG: Unstructured
console.log('Processing request');

// ✅ CORRECT: Structured
console.log(`[${requestId}] Processing request`);
```


## Dashboard Refactoring Patterns

### Finding Duplicate Code

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brendadeeznuts1111) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
