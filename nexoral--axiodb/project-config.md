---
trigger: always_on
description: **AxioDB** is an embedded NoSQL database for Node.js with MongoDB-style queries. Pure TypeScript/JavaScript, zero native dependencies.
---

# AxioDB Core Rules for Cursor IDE

## Project Context

**AxioDB** is an embedded NoSQL database for Node.js with MongoDB-style queries. Pure TypeScript/JavaScript, zero native dependencies.

## Mandatory Workflows

### After EVERY Code Change
```bash
npm run build  # MANDATORY - catch TypeScript errors immediately
```

### For ANY Feature Change
1. Update tests in `Test/modules/`
2. Run `npm test` - all tests must pass
3. Update docs (README.md, Document/, Dockerfile)
4. Run `npm run lint` - must pass

## Definition of "Done"

A task is NOT complete until ALL of these are true:
- ✅ Code written following standards
- ✅ `npm run build` passes with zero errors
- ✅ Tests added/updated in `Test/modules/`
- ✅ `npm test` passes - all tests green
- ✅ `npm run lint` passes
- ✅ Documentation updated (README, Document/, Dockerfile)
- ✅ No breaking changes (or explicitly noted and approved)
- ✅ Self-reviewed for performance, security, maintainability

## Architecture Constraints

### Singleton Pattern (NON-NEGOTIABLE)
```typescript
export class AxioDB {
  private static _instance: AxioDB;

  constructor() {
    if (AxioDB._instance) {
      throw new Error("Only one instance of AxioDB is allowed.");
    }
    AxioDB._instance = this;
  }
}
```

**Critical Implication**: Tests MUST run in separate child processes. Cannot run tests in parallel due to singleton constraint.

### File-Per-Document Storage
```
{RootPath}/{DatabaseName}/{CollectionName}/{documentId}.axiodb
```

### Dual-Write Pattern (Indexes)
```typescript
// ALWAYS write to BOTH memory AND disk
await this.indexCache.set(key, data, TTL);  // Memory (speed)
await this.fileManager.writeFile(path, JSON.stringify(data));  // Disk (durability)
```

### Random Cache TTL (5-15 minutes)
```typescript
const TTL = Math.floor(Math.random() * (15 - 5 + 1) + 5) * 60 * 1000;
```
**Why**: Prevents cache stampede/thundering herd

## TypeScript Standards (STRICT)

### NO `any` Types - EVER
```typescript
// ❌ ABSOLUTELY FORBIDDEN
const result: any = await operation();

// ✅ REQUIRED
interface OperationResult {
  success: boolean;
  data: DocumentData;
}
const result: OperationResult = await operation();
```

### Strict Null Checks
```typescript
// ✅ GOOD
function get(id: string): Document | null {
  return this.cache.get(id) ?? null;
}

const doc = get('123');
if (doc !== null) {
  console.log(doc.name);
}
```

## SOLID Principles

### Single Responsibility
Each class/module has ONE reason to change.

### Don't Repeat Yourself (DRY)
If same logic appears in 2+ files, extract to `Helper/` directory.

### No Hacky Solutions
```typescript
// ❌ FORBIDDEN
setTimeout(() => { /* hope this works */ }, 1000);
eval(userInput);
try { risky(); } catch (e) { /* ignore */ }

// ✅ REQUIRED
await properAsyncOperation();
const sanitized = validateAndSanitize(userInput);
try { await risky(); } catch (error) {
  logger.error('Operation failed', error);
  return ResponseHelper.error('Failed', StatusCodes.ERROR);
}
```

## Testing Requirements

### Location
```
Test/modules/
├── crud.test.js          # CRUD operations
├── transaction.test.js   # Transactions, WAL, savepoints
└── read.test.js          # Read optimizations, caching
```

### Coverage Required
- Happy path (success scenarios)
- Edge cases (empty, null, undefined, large data)
- Error cases (validation failures, file errors, conflicts)
- Concurrent operations
- Backward compatibility

### Test Execution
```bash
npm test                           # All tests
npm test crud                      # CRUD tests only
node Test/modules/crud.test.js     # Direct execution
```

## Documentation Requirements

### README.md
Update when:
- New public API methods
- Feature additions
- Behavior changes
- Configuration changes

### Document/ (React Docs Site)
```bash
cd Document
npm run dev    # localhost:5173
# Edit src/pages/
npm run build  # Verify builds
```

Update for ALL new features with:
- Feature overview
- Code examples (tested!)
- API reference
- Best practices
- Common pitfalls

### Dockerfile
Update when:
- Port numbers change
- Environment variables added
- Build process changes
- Startup command changes

### JSDoc Comments
Required for ALL public methods:
```typescript
/**
 * Method description.
 *
 * @param {Type} paramName - Description
 * @returns {Promise<Type>} Description
 * @throws {Error} When something fails
 *
 * @example
 * const result = await method(param);
 */
```

## Performance Standards

### 1. Use InMemoryCache
```typescript
const cached = this.cache.get(key);
if (cached) return cached;

const data = await this.readFromDisk(id);
this.cache.set(key, data, TTL);
return data;
```

### 2. Batch Operations
```typescript
// ✅ PARALLEL
await Promise.all(docs.map(d => this.insert(d)));

// ❌ SEQUENTIAL (slow)
for (const d of docs) { await this.insert(d); }
```

### 3. Use Map for O(1) Lookups
```typescript
// ✅ O(1)
const map = new Map<string, Document>();
const found = map.get(id);

// ❌ O(n)
const found = array.find(d => d.id === id);
```

### 4. Avoid Unnecessary File I/O
```typescript
// ✅ Read once
const docs = await this.loadAll();
const filtered = docs.filter(d => d.age > 25);

// ❌ Multiple reads
for (const id of ids) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nexoral/AxioDB](https://github.com/nexoral/AxioDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
