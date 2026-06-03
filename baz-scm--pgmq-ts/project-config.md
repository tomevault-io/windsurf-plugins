---
trigger: always_on
description: PGMQ-TS is a TypeScript wrapper for PostgreSQL-based message queues (PGMQ). It provides a type-safe interface for creating and managing message queues backed by PostgreSQL.
---

# PGMQ-TS Architecture Guide

## Project Overview
PGMQ-TS is a TypeScript wrapper for PostgreSQL-based message queues (PGMQ). It provides a type-safe interface for creating and managing message queues backed by PostgreSQL.

**Key Concepts:**
- Messages are stored in PostgreSQL tables with a `q_` prefix (e.g., `pgmq.q_my_queue`)
- Each queue has a matching archive table with an `a_` prefix (e.g., `pgmq.a_my_queue`)
- Visibility timeout (vt) controls when messages become available for reading
- Messages can be read, deleted, or archived

## File Structure

```
src/
├── index.ts                  # Main entry point, exports PGMQ and Queue
├── classes/
    ├── pgmq.ts              # Main PGMQ class with connection pool
    ├── queue.ts             # Queue-specific operations
    ├── types.ts             # Type definitions and parsers
    ├── queries.ts           # SQL query builders
    └── utils.ts             # Utility functions (transactions)
```

## Core Architecture

### 1. **Entry Point** (`src/index.ts`)
- Exports the `PGMQ` class (renamed from `Pgmq`)
- Exports the `Queue` class for type referencing

### 2. **PGMQ Class** (`src/classes/pgmq.ts`)
Main orchestrator class that:
- Manages PostgreSQL connection pool
- Provides schema operations (`createSchema`, `deleteSchema`)
- Provides queue operations (`createQueue`, `deleteQueue`, `getQueue`)
- Provides message operations (`sendMessage`, `readMessage`, `deleteMessage`, `archiveMessage`)
- Validates queue names (alphanumeric + underscore, max 47 chars)

**Connection Pattern:**
```typescript
// Most operations:
const connection = await this.pool.connect()
await connection.query(query)
connection.release()

// Transactional operations (readMessage):
const result = await executeQueryWithTransaction(this.pool, query)
```

### 3. **Queue Class** (`src/classes/queue.ts`)
Focused interface for queue-specific operations:
- Stores queue name and pool reference
- Provides the same message operations as PGMQ but without requiring queue name
- Methods: `readMessage`, `deleteMessage`, `archiveMessage`

**Pattern:** All methods mirror PGMQ methods but use `this.name` instead of accepting a queue parameter.

### 4. **Types** (`src/classes/types.ts`)
Defines core data structures:
- `Message<T>`: Public-facing message type with camelCase properties
- `DbMessage`: Internal database result type with snake_case properties
- `parseDbMessage<T>()`: Converts DB format to public format

**Message Structure:**
```typescript
interface Message<T> {
  msgId: number           // Unique message ID
  readCount: number       // How many times message was read
  enqueuedAt: Date       // When message was added
  vt: Date               // Visibility timeout
  message: T             // Actual payload (generic)
}
```

### 5. **Queries** (`src/classes/queries.ts`)
SQL query builders for all operations:
- Schema: `createSchemaQuery()`, `deleteSchemaQuery()`
- Queue: `createQueueQuery(name)`, `deleteQueueQuery(name)`
- Message: `sendQuery(queue, vt)`, `readQuery(queue, vt)`, `deleteQuery(queue, id)`, `archiveQuery(queue, id)`

**Constants:**
```typescript
const PGMQ_SCHEMA = "pgmq"
const QUEUE_PREFIX = "q"
const ARCHIVE_PREFIX = "a"
```

### 6. **Utils** (`src/classes/utils.ts`)
Transaction management:
- `executeQueryWithTransaction(pool, query)`: Wraps query in BEGIN/COMMIT/ROLLBACK
- Used for operations requiring atomicity (currently only `readMessage`)

## Adding New Methods (Pattern Guide)

### Step-by-Step Process for Adding Methods Like `readMessage` or `deleteMessage`

#### Step 1: Add Query Builder (`src/classes/queries.ts`)
```typescript
export function newMethodQuery(queue: string, param: type) {
  return `SQL QUERY HERE
          USING ${PGMQ_SCHEMA}.${QUEUE_PREFIX}_${queue}
          RETURNING relevant_fields;`
}
```

**Checklist:**
- [ ] Use `PGMQ_SCHEMA` constant for schema name
- [ ] Use `QUEUE_PREFIX` for queue table or `ARCHIVE_PREFIX` for archive table
- [ ] Add `RETURNING` clause if you need to return data
- [ ] Use parameterized queries ($1, $2) for user input to prevent SQL injection
- [ ] Consider if operation should be in a transaction

#### Step 2: Add Method to PGMQ Class (`src/classes/pgmq.ts`)

**Import the query:**
```typescript
import { newMethodQuery } from "./queries"
```

**Add the method:**
```typescript
/**
 * Clear description of what this method does
 * @param queue - the name of the queue
 * @param param - description of parameter
 * @return what the method returns
 */
public async newMethod(queue: string, param: type): Promise<ReturnType> {
  // Pattern 1: Simple query (no transaction needed)
  const connection = await this.pool.connect()
  const query = newMethodQuery(queue, param)
  const result = await connection.query(query)
  connection.release()
  return processResult(result)

  // Pattern 2: Transactional query (for operations that must be atomic)
  const query = newMethodQuery(queue, param)
  const result = await executeQueryWithTransaction(this.pool, query)
  return processResult(result)
}
```

**Decision Guide - Transaction vs Simple:**
- Use **transaction** (`executeQueryWithTransaction`) when:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baz-scm/pgmq-ts](https://github.com/baz-scm/pgmq-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
