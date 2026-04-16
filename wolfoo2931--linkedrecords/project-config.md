---
trigger: always_on
description: **LinkedRecords** is a Backend-as-a-Service (BaaS) that provides a NoSQL database you can connect to directly from single-page applications - no backend code required.
---

# LinkedRecords - Developer Guide for Claude

## Project Overview

**LinkedRecords** is a Backend-as-a-Service (BaaS) that provides a NoSQL database you can connect to directly from single-page applications - no backend code required.

- **Repository:** https://github.com/wolfoo2931/linkedrecords
- **License:** MIT
- **Version:** 0.1.0
- **Author:** Oliver Wolf

## Core Concept

You can think of LinkedRecords as a bucket where anyone can sign up and insert data. As long as you don't share this data with other users or groups, only you can access what you've written.

Key idea: Instead of defining universal authorization rules in the backend, the user who inserts a data record specifies who can read it.

## Tech Stack

### Backend
- **TypeScript** on Node.js v24
- **Express.js** for HTTP API
- **PostgreSQL** (triplestore for facts + attribute storage)
- **Socket.IO** with Redis Streams for real-time
- **OpenID Connect** (OIDC) for authentication
- **MinIO/S3** for blob storage (optional)
- **Redis** for caching/pub-sub

### Frontend SDK
- **TypeScript** browser SDK
- **oidc-client-ts** for OIDC
- **Socket.IO client** for real-time

### Testing
- **Mocha + Chai** for unit tests
- **WebdriverIO v9** for E2E tests

## Project Structure

```
src/
├── server/                  # Express server
│   ├── controllers/         # API endpoints
│   │   ├── facts.ts        # Fact management (triplestore)
│   │   ├── attributes.ts   # Attribute CRUD
│   │   ├── quota.ts        # Storage quotas
│   │   └── userinfo.ts     # User info
│   ├── middleware/         # Auth, error handling
│   ├── payment_provider/   # Paddle integration
│   └── quota/             # Quota management
├── browser_sdk/            # Client SDK
├── attributes/             # Data storage
│   ├── key_value/         # JSON documents (CRDT)
│   ├── long_text/         # Large text (OT)
│   ├── blob/              # Binary files
│   └── attribute_storage/ # PostgreSQL/S3 backends
└── facts/                 # Authorization system
    ├── server/            # Fact management
    └── client/            # Fact queries

lib/                       # Shared utilities
├── client-server-bus/    # WebSocket/HTTP layer
├── pg-log/               # PostgreSQL with logging
└── utils/                # Helpers

specs.wdio/               # Integration tests
├── tinytodo/             # Example todo app
├── testapp/              # Test application
└── helpers/              # Test utilities
```

## Key Concepts

### 1. Facts (Triples)

LinkedRecords uses a **triplestore pattern**: `(subject, predicate, object)`

Facts represent relationships between entities. They are stored in PostgreSQL and control authorization.

### 2. Terms - MUST Be Declared First

**CRITICAL:** Before using a term like "TodoList" or "Organization", you **MUST declare it** using `$isATermFor`:

```typescript
// Declare terms before using them
await client.Fact.createAll([
  ['Organization', '$isATermFor', 'A business organization'],
  ['TodoList', '$isATermFor', 'A list of tasks'],
  ['AdminTeam', '$isATermFor', 'Team of administrators'],
]);

// Now you can use these terms
const org = await client.Attribute.createKeyValue(
  { name: 'Acme Inc' },
  [['$it', 'isA', 'Organization']]  // 'Organization' was declared above
);
```

**Terms are public domain** - once declared, anyone can refer to them in facts.

### 3. Reserved Predicates (Starting with $)

**Authorization predicates** (start with `$`):
- `$isATermFor` - Define terms (must be declared first)
- `$isAccountableFor` - Ownership/accountability (auto-assigned to creator)
- `$isMemberOf` - Membership in groups/teams
- `$isHostOf` - Can add members to a group
- `$canRead` - Read-only access
- `$canAccess` - Read and write access
- `$canRefine` - Can use as subject in facts (conceptor permission)
- `$canReferTo` - Can use as object in facts (referrer permission)

**System predicates**:
- `$hasDataType` - Filter by attribute type (KeyValueAttribute, LongTextAttribute, BlobAttribute)
- `$latest(predicate)` - Get latest value for a predicate
- `$not(value)` - Negation

**Custom predicates** (no `$` prefix):
- `isA` - Type classification
- `stateIs` - Current state
- `belongsTo` - Belongs to relationship
- Any custom predicates you define

### 4. Accountability

- When you create an attribute, you're **automatically accountable** for it
- A fact `[userId, '$isAccountableFor', attributeId]` is created automatically
- Accountability determines quota usage
- You can transfer accountability: `[orgId, '$isAccountableFor', '$it']`

### 5. Attribute Types

#### Key-Value Attributes (CRDT)
```typescript
const attr = await client.Attribute.createKeyValue(
  { name: 'value', nested: { data: 123 } },
  [['$it', 'isA', 'MyType']]
);
```

#### Long Text Attributes (OT)
```typescript
const attr = await client.Attribute.create('longText', 'Initial text content');
```

#### Blob Attributes
```typescript
const attr = await client.Attribute.create('blob', binaryData);
```

### 6. Creating Attributes

**Simple creation:**
```typescript
const list = await client.Attribute.createKeyValue(
  { title: 'Shopping', items: [] },
  [['$it', 'isA', 'TodoList']]
);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wolfoo2931) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
