---
trigger: always_on
description: Testing patterns with Vitest and @sim/testing
---



# Testing Patterns

Use Vitest. Test files: `feature.ts` → `feature.test.ts`

## Global Mocks (vitest.setup.ts)

These modules are mocked globally — do NOT re-mock them in test files unless you need to override behavior:

- `@sim/db` → `databaseMock`
- `@sim/db/schema` → `schemaMock`
- `drizzle-orm` → `drizzleOrmMock`
- `@sim/logger` → `loggerMock`
- `@/lib/auth` → `authMock`
- `@/lib/auth/hybrid` → `hybridAuthMock` (with default session-delegating behavior)
- `@/lib/core/utils/request` → `requestUtilsMock`
- `@/stores/console/store`, `@/stores/terminal`, `@/stores/execution/store`
- `@/blocks/registry`
- `@trigger.dev/sdk`

## Structure

```typescript
/**
 * @vitest-environment node
 */
import { createMockRequest } from '@sim/testing'
import { beforeEach, describe, expect, it, vi } from 'vitest'

const { mockGetSession } = vi.hoisted(() => ({
  mockGetSession: vi.fn(),
}))

vi.mock('@/lib/auth', () => ({
  auth: { api: { getSession: vi.fn() } },
  getSession: mockGetSession,
}))

import { GET, POST } from '@/app/api/my-route/route'

describe('my route', () => {
  beforeEach(() => {
    vi.clearAllMocks()
    mockGetSession.mockResolvedValue({ user: { id: 'user-1' } })
  })

  it('returns data', async () => {
    const req = createMockRequest('GET')
    const res = await GET(req)
    expect(res.status).toBe(200)
  })
})
```

## Performance Rules (Critical)

### NEVER use `vi.resetModules()` + `vi.doMock()` + `await import()`

This is the #1 cause of slow tests. It forces complete module re-evaluation per test.

```typescript
// BAD — forces module re-evaluation every test (~50-100ms each)
beforeEach(() => {
  vi.resetModules()
  vi.doMock('@/lib/auth', () => ({ getSession: vi.fn() }))
})
it('test', async () => {
  const { GET } = await import('./route')  // slow dynamic import
})

// GOOD — module loaded once, mocks reconfigured per test (~1ms each)
const { mockGetSession } = vi.hoisted(() => ({
  mockGetSession: vi.fn(),
}))
vi.mock('@/lib/auth', () => ({ getSession: mockGetSession }))
import { GET } from '@/app/api/my-route/route'

beforeEach(() => { vi.clearAllMocks() })
it('test', () => {
  mockGetSession.mockResolvedValue({ user: { id: '1' } })
})
```

**Only exception:** Singleton modules that cache state at module scope (e.g., Redis clients, connection pools). These genuinely need `vi.resetModules()` + dynamic import to get a fresh instance per test.

### NEVER use `vi.importActual()`

This defeats the purpose of mocking by loading the real module and all its dependencies.

```typescript
// BAD — loads real module + all transitive deps
vi.mock('@/lib/workspaces/utils', async () => {
  const actual = await vi.importActual('@/lib/workspaces/utils')
  return { ...actual, myFn: vi.fn() }
})

// GOOD — mock everything, only implement what tests need
vi.mock('@/lib/workspaces/utils', () => ({
  myFn: vi.fn(),
  otherFn: vi.fn(),
}))
```

### Mock heavy transitive dependencies

If a module under test imports `@/blocks` (200+ files), `@/tools/registry`, or other heavy modules, mock them:

```typescript
vi.mock('@/blocks', () => ({
  getBlock: () => null,
  getAllBlocks: () => ({}),
  getAllBlockTypes: () => [],
  registry: {},
}))
```

### Use `@vitest-environment node` unless DOM is needed

Only use `@vitest-environment jsdom` if the test uses `window`, `document`, `FormData`, or other browser APIs. Node environment is significantly faster.

### Avoid real timers in tests

```typescript
// BAD
await new Promise(r => setTimeout(r, 500))

// GOOD — use minimal delays or fake timers
await new Promise(r => setTimeout(r, 1))
// or
vi.useFakeTimers()
```

## Centralized Mocks (prefer over local declarations)

`@sim/testing` exports ready-to-use mock modules for common dependencies. Import and pass directly to `vi.mock()` — no `vi.hoisted()` boilerplate needed. Each paired `*MockFns` object exposes the underlying `vi.fn()`s for per-test overrides.

| Module mocked | Import | Factory form |
|---|---|---|
| `@/app/api/auth/oauth/utils` | `authOAuthUtilsMock`, `authOAuthUtilsMockFns` | `vi.mock('@/app/api/auth/oauth/utils', () => authOAuthUtilsMock)` |
| `@/app/api/knowledge/utils` | `knowledgeApiUtilsMock`, `knowledgeApiUtilsMockFns` | `vi.mock('@/app/api/knowledge/utils', () => knowledgeApiUtilsMock)` |
| `@/app/api/workflows/utils` | `workflowsApiUtilsMock`, `workflowsApiUtilsMockFns` | `vi.mock('@/app/api/workflows/utils', () => workflowsApiUtilsMock)` |
| `@sim/audit` | `auditMock`, `auditMockFns` | `vi.mock('@sim/audit', () => auditMock)` |
| `@/lib/auth` | `authMock`, `authMockFns` | `vi.mock('@/lib/auth', () => authMock)` |
| `@/lib/auth/hybrid` | `hybridAuthMock`, `hybridAuthMockFns` | `vi.mock('@/lib/auth/hybrid', () => hybridAuthMock)` |
| `@/lib/copilot/request/http` | `copilotHttpMock`, `copilotHttpMockFns` | `vi.mock('@/lib/copilot/request/http', () => copilotHttpMock)` |
| `@/lib/core/config/env` | `envMock`, `createEnvMock(overrides)` | `vi.mock('@/lib/core/config/env', () => envMock)` |
| `@/lib/core/config/feature-flags` | `featureFlagsMock` | `vi.mock('@/lib/core/config/feature-flags', () => featureFlagsMock)` |
| `@/lib/core/config/redis` | `redisConfigMock`, `redisConfigMockFns` | `vi.mock('@/lib/core/config/redis', () => redisConfigMock)` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simstudioai/sim](https://github.com/simstudioai/sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
