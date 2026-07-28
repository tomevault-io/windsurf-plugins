---
trigger: always_on
description: cnpmcore is a TypeScript-based private NPM registry implementation built with Egg.js framework. It provides enterprise-grade package management with support for MySQL/PostgreSQL databases, Redis caching, and optional Elasticsearch.
---

# cnpmcore - Private NPM Registry for Enterprise

cnpmcore is a TypeScript-based private NPM registry implementation built with Egg.js framework. It provides enterprise-grade package management with support for MySQL/PostgreSQL databases, Redis caching, and optional Elasticsearch.

**ALWAYS reference these instructions first** and fallback to search or bash commands only when you encounter unexpected information that does not match the information here.

## Code Style and Conventions

### Linting and Formatting

The project uses **Vite+** (the `vp` CLI) as the unified toolchain. Lint/format config lives in `vite.config.ts` (`lint` and `fmt` blocks) — there are no standalone `.oxlintrc.json`/`.oxfmtrc.json` files (running `oxlint`/`oxfmt` directly would NOT pick up project rules; always go through `vp`).

- **Linter**: Oxlint via `vp lint` (fast Rust-based linter) with type-aware checking
- **Formatter**: Oxfmt via `vp fmt` (sole formatter, no Prettier)
- **Unified check**: `vp check` runs format + lint + type-check in one pass (`vp check --fix` auto-fixes) — the preferred validation loop
- **Pre-commit hooks**: Vite+ git hooks installed by `vp config` (the `prepare` script); `.vite-hooks/pre-commit` runs `vp staged`, which applies `vp check --fix` to staged files (replaces the old Husky + lint-staged setup)

**Code Style Rules** (configured in `vite.config.ts`):

```javascript
// fmt block
{
  "singleQuote": true,        // Use single quotes
  "printWidth": 120           // 120 character line width
  // 2-space indentation and ES5 trailing commas are oxfmt defaults
}

// lint block (rules)
{
  "max-params": ["error", 6], // Maximum 6 function parameters
  "no-console": "warn",        // Warn on console usage
  "import/no-anonymous-default-export": "error"
}
```

**Linting Commands:**

```bash
npm run lint         # Check for linting errors (= vp lint)
npm run lint:fix     # Auto-fix linting issues (= vp lint --fix)
npm run typecheck    # TypeScript type checking without build (tsc --noEmit)
vp check             # Format + lint + type-check in one pass (add --fix to auto-fix)
```

### TypeScript Conventions

- Use strict TypeScript with comprehensive type definitions
- Avoid `any` types - use proper typing or `unknown`
- Export types and interfaces for reusability
- Use ES modules (`import/export`) syntax throughout

### Testing Conventions

- Test files use `.test.ts` suffix
- Use `@eggjs/mock` for mocking and testing
- Tests organized to mirror source structure in `test/` directory
- Use `assert` from `node:assert/strict` for assertions
- Mock external dependencies using `mock()` from `@eggjs/mock`

**Test Infrastructure**:

- `test/.setup.ts` - Global beforeEach/afterEach hooks
- `test/TestUtil.ts` - Comprehensive test utilities
- `test/fixtures/` - Mock data and responses

**Key TestUtil Methods**:

- `TestUtil.createUser(options)` - Create test user with auth tokens
- `TestUtil.createPackage(options)` - Create full package in system
- `TestUtil.getFullPackage(options)` - Get mock package JSON
- `TestUtil.truncateDatabase()` - Clear all tables between tests
- `TestUtil.query(sql)` - Execute raw SQL (MySQL/PostgreSQL)

**Mocking Patterns**:

```typescript
// Config mocking
mock(app.config.cnpmcore, 'propertyName', newValue);

// HTTP mocking
app.mockHttpclient('https://example.com/path', 'GET', {
  data: await TestUtil.readFixturesFile('fixture.json'),
  persist: false,
});

// Log assertions
app.mockLog();
await someOperation();
app.expectLog(/pattern/);
```

**Getting DI Objects in Tests**:

```typescript
const service = await app.getEggObject(PackageManagerService);
```

**Scheduled Task Testing**:

```typescript
await app.runSchedule(SyncPackageWorkerPath);
```

**Test Naming Pattern:**

```typescript
describe('test/path/to/SourceFile.test.ts', () => {
  describe('[HTTP_METHOD /api/path] functionName()', () => {
    it('should handle expected behavior', async () => {
      // Test implementation
    });
  });
});
```

## Domain-Driven Design (DDD) Architecture

cnpmcore follows **Domain-Driven Design** principles with clear separation of concerns:

### Layer Architecture (Dependency Flow)

```
Controller (HTTP Interface Layer)
    ↓ depends on
Service (Business Logic Layer)
    ↓ depends on
Repository (Data Access Layer)
    ↓ depends on
Model (ORM/Database Layer)

Entity (Domain Models - no dependencies, pure business logic)
Common (Utilities and Adapters - available to all layers)
```

### Layer Responsibilities

**Controller Layer** (`app/port/controller/`):

- HTTP request/response handling
- Request validation using `@eggjs/typebox-validate`
- User authentication and authorization
- **NO business logic** - delegate to Services
- Inheritance: `YourController extends AbstractController extends MiddlewareController`
- Auto-applied middlewares: `AlwaysAuth`, `Tracing`, `ErrorHandler`

Key `AbstractController` methods:

- `ensurePublishAccess(ctx, fullname)` - Authorization check for package publish
- `getPackageEntity(scope, name)` - Fetch package with error handling
- `setCDNHeaders(ctx)` - Set cache control headers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cnpm/cnpmcore](https://github.com/cnpm/cnpmcore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
