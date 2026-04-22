---
trigger: always_on
description: Backend-as-a-Service (BaaS) with optimistic updates, multiplayer, offline support, and relational data. Firebase alternative with relations.
---

# What is InstantDB

Backend-as-a-Service (BaaS) with optimistic updates, multiplayer, offline support, and relational data. Firebase alternative with relations.

## Core Features

* Optimistic updates
* Real-time multiplayer sync
* Offline-first architecture
* Relational data support
* Web and mobile compatible

## API

CRITICAL: These are the only APIs from the react package you need to know. do not hallucinate other APIs.

```
// Initialization
init<Schema>(config: InstantConfig<Schema>): InstantReactWebDatabase<Schema>

// Transaction builder
tx: TxChunk<Schema>
id(): string
lookup(attribute: string, value: any): Lookup

// Schema builder
i.schema({ entities, links?, rooms? })
i.entity(attrs)
i.string(), i.number(), i.boolean(), i.date(), i.json(), i.any()

// Core Database Methods (on db instance)
db.transact(chunks)

// React Hooks (on db instance)
db.useQuery(query, opts?)
db.useAuth()
db.room(type?, id?)

// Auth Methods (on db.auth)
db.auth.sendMagicCode({ email })
db.auth.signInWithMagicCode({ email, code })
db.auth.signOut(opts?)

// Room Hooks (on db.rooms) - IMPORTANT: These are called on db.rooms, not on room instances
db.rooms.useTopicEffect(room, topic, onEvent)
db.rooms.usePublishTopic(room, topic) // returns: (data) => void
db.rooms.usePresence(room, opts?) // returns: { peers, user, publishPresence, isLoading }
db.rooms.useSyncPresence(room, data, deps?)
db.rooms.useTypingIndicator(room, inputName, opts?) // returns: { active, setActive, inputProps }

// Components
<Cursors room={room} {...props} />
```

# How to initialize DB

Create a central DB instance (single connection maintained per app ID):

```typescript
// lib/db.ts
import { init } from '@instantdb/react';
import schema from '../instant.schema';

export const db = init({
  // Get your app ID from https://instantdb.com
  appId: 'your-app-id',
  schema
});
```

`init` accepts the following parameters:

```typescript
export type InstantConfig<S extends InstantSchemaDef<any, any, any>> = {
  appId: string;
  schema?: S;
};
```

# How to do queries

## Core Concepts
- **Namespaces**: Entity collections (tables)
- **Queries**: JS objects describing data needs
- **Associations**: Entity relationships

## Query Structure
```typescript
{
  namespace1: {
    $: { /* operators */ },
    linkedNamespace: { $: { /* operators */ } }
  },
  namespace2: { /* ... */ }
}
```

## Basic Usage

**Required**: Handle `isLoading` and `error` states:
```typescript
const { isLoading, data, error } = db.useQuery({ todos: {} })
if (isLoading) return
if (error) return (<div>Error: {error.message}</div>)
return <pre>{JSON.stringify(data, null, 2)}</pre>
```

### Fetch Operations
```typescript
// Single namespace
const query = { goals: {} }

// Multiple namespaces
const query = { goals: {}, todos: {} }
```

## Filtering

### By ID
```typescript
const query = {
  goals: {
    $: { where: { id: 'goal-1' } }
  }
}
```

### Multiple Conditions (AND)
```typescript
const query = {
  todos: {
    $: { where: { completed: true, priority: 'high' } }
  }
}
```

## Associations (JOINs)

### Fetch Related
```typescript
// Goals with todos
const query = { goals: { todos: {} } }

// Inverse: Todos with goals
const query = { todos: { goals: {} } }
```

### Filter by Association
```typescript
// Dot notation for associated values
const query = {
  goals: {
    $: { where: { 'todos.title': 'Go running' } },
    todos: {}
  }
}
```

### Filter Associated Entities
```typescript
const query = {
  goals: {
    todos: {
      $: { where: { completed: true } }
    }
  }
}
```

## Operators

### Logical
```typescript
// AND
where: { and: [{ 'todos.priority': 'high' }, { 'todos.dueDate': { $lt: tomorrow } }] }

// OR
where: { or: [{ priority: 'high' }, { dueDate: { $lt: tomorrow } }] }
```

### Comparison (indexed fields only)
- `$gt`, `$lt`, `$gte`, `$lte`
```typescript
where: { timeEstimate: { $gt: 2 } }
```

### Other Operators
```typescript
// IN
where: { priority: { $in: ['high', 'critical'] } }

// NOT
where: { location: { $not: 'work' } }

// NULL check
where: { location: { $isNull: true } }

// Pattern matching (indexed strings)
where: { title: { $like: 'Get%' } }     // Case-sensitive
where: { title: { $ilike: 'get%' } }    // Case-insensitive
```

Pattern syntax:
- `'prefix%'` - Starts with
- `'%suffix'` - Ends with
- `'%substring%'` - Contains

## Pagination & Ordering

### Pagination (top-level only)
```typescript
$: { limit: 10, offset: 10 }
```

### Ordering (indexed fields)
```typescript
$: { order: { dueDate: 'asc' } }  // or 'desc'
```

## Field Selection
```typescript
// Select specific fields
$: { fields: ['title', 'status'] }

// With nested associations
goals: {
  $: { fields: ['title'] },
  todos: { $: { fields: ['status'] } }
}
```

## Deferred Queries
```typescript
const query = user ? { todos: { $: { where: { userId: user.id } } } } : null
```

## Complex Example
```typescript
const query = {
  goals: {
    $: {
      where: { or: [{ status: 'active' }, { 'todos.priority': 'high' }] },
      limit: 5,
      order: { serverCreatedAt: 'desc' },
      fields: ['title', 'description']
    },
    todos: {
      $: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Benanna2019) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
