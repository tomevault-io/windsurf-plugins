---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- v2.1.1 -->

## Package

`@djpanda/convex-authz` — a Convex component providing RBAC/ABAC/ReBAC authorization with O(1) indexed lookups (inspired by Google Zanzibar). Published as a Convex component via `defineComponent("authz")`.

## Commands

```bash
npm test                 # Run vitest (run mode + type checking)
npm run test:watch       # Vitest in watch mode
npm run build            # Compile via tsconfig.build.json → dist/
npm run build:clean      # Remove dist + tsbuildinfo, full codegen rebuild
npm run build:codegen    # Generate Convex component code + rebuild
npm run lint             # ESLint on all files
npm run typecheck        # tsc --noEmit across src, example, and example/convex
npm run dev              # Parallel: convex dev + vite (example app) + build watcher
```

Run a single test file: `npx vitest run src/component/queries.test.ts`
Run a single test by name: `npx vitest run -t "test name pattern"`
Debug tests: `npm run test:debug` (enables Node inspector, no file parallelism).

## Architecture

### Unified Architecture (v2)

One `Authz` class provides O(1) reads, ABAC policy support, and ReBAC — all in one.

**Dual-layer design:** Source tables store ground truth; effective tables store pre-computed O(1) lookups. All writes go to BOTH layers via `unified.ts` mutations.

- **Source tables**: `roleAssignments`, `userAttributes`, `permissionOverrides`, `relationships`, `auditLog`
- **Effective tables**: `effectivePermissions`, `effectiveRoles`, `effectiveRelationships`

**Permission check (`can()`) tiered resolution:**
1. O(1) exact lookup in `effectivePermissions` (covers RBAC + overrides)
2. If `policyResult == "deferred"` → evaluate ABAC policy at read time
3. Wildcard pattern fallback for `docs:*` style permissions

**ABAC policy classification:**
- **Static** (`type: "static"`) — evaluated at write time, result stored in `effectivePermissions.policyResult`
- **Deferred** (`type: "deferred"`) — evaluated at read time via `canWithContext()`

**Three authorization models** (RBAC, ABAC, ReBAC) all available on the single `Authz` class. `IndexedAuthz` is a deprecated alias.

### Scope System

Scope (`{ type: string; id: string }`) enables resource-level permissions. A role/permission can be global (no scope) or scoped to a resource (e.g., `{ type: "team", id: "team_123" }`). Indexed tables use `scopeKey` field: `"global"` or `"type:id"`.

### Key File Map

- `src/component/schema.ts` — 8 tables with all indexes
- `src/component/unified.ts` — **v2 core**: tiered checkPermission query + dual-write mutations (assignRoleUnified, revokeRoleUnified, grantPermissionUnified, denyPermissionUnified, addRelationUnified, removeRelationUnified, setAttributeWithRecompute, recomputeUser)
- `src/component/mutations.ts` — source-table mutations (offboardUser, deprovisionUser, cleanup, audit)
- `src/component/queries.ts` — read queries (getUserRoles, hasRole, getUserAttributes, getAuditLog). `checkPermission`/`checkPermissions` are now internal.
- `src/component/indexed.ts` — O(1) read queries (checkPermissionFast, hasRoleFast, hasRelationFast, getUserPermissionsFast, getUserRolesFast). Write mutations are now internal.
- `src/component/rebac.ts` — relationship traversal (checkRelationWithTraversal, listAccessibleObjects, listUsersWithAccess)
- `src/component/helpers.ts` — `matchesPermissionPattern`, scope matching, policy context
- `src/client/index.ts` — unified `Authz` class + `definePermissions`, `defineRoles`, `definePolicies`, `defineTraversalRules`, `defineRelationPermissions`, `defineCaveats` helpers. `IndexedAuthz` is a deprecated alias.
- `src/client/validation.ts` — input validation for client methods
- `src/react/index.ts` — `AuthzProvider`, `useCanUser`, `useUserRoles`, `PermissionGate`

### Package Exports

- `.` → `dist/client/index.js` (unified Authz class, define* helpers, deprecated IndexedAuthz alias)
- `./react` → `dist/react/index.js` (React hooks/components)
- `./convex.config` → `dist/component/convex.config.js` (component registration)

### Type-Safe Permission/Role Definitions

`definePermissions()` and `defineRoles(permissions, ...)` use generics so that role definitions are type-checked against declared permissions. Roles support `inherits` (single parent) and `includes` (multiple roles) with cycle detection via `flattenRolePermissions()`.

### Wildcard Permissions

Permission strings support patterns: `"*"` (all), `"resource:*"` (all actions on resource), `"*:action"` (action on all resources). Matching happens in `matchesPermissionPattern()`.

## Test Pattern

Tests use `convex-test`:

```typescript
import { convexTest } from "convex-test";
import schema from "./schema.js";
import { api } from "./_generated/api.js";

const t = convexTest(schema, import.meta.glob("./**/*.ts"));
await t.mutation(api.mutations.assignRole, { userId, role, ... });
const result = await t.query(api.queries.hasRole, { userId, role, ... });
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbjpanda/convex-authz](https://github.com/dbjpanda/convex-authz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
