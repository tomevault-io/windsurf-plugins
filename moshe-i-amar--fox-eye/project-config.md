---
trigger: always_on
description: Fox-Eye (GeoMap) is a real-time operational tracking and geofencing app for hierarchical military-style organizations. Users share live location on a Leaflet map; admins define Areas of Operations (AOs) as GeoJSON polygons; the system detects breaches and fires real-time alerts.
---

# Fox-Eye — Claude Code Context

## What This App Does
Fox-Eye (GeoMap) is a real-time operational tracking and geofencing app for hierarchical military-style organizations. Users share live location on a Leaflet map; admins define Areas of Operations (AOs) as GeoJSON polygons; the system detects breaches and fires real-time alerts.

## Monorepo Structure
```
fox-eye/
├── client/     React 18 + Vite + Tailwind + Leaflet (ESM)
├── server/     Node.js + Express + MongoDB + Socket.IO (CommonJS)
└── .claude/    Claude Code config and commands
```

---

## Server (server/src/)

### Stack
- Node.js, **CommonJS** (`require`/`module.exports` — never use `import/export`)
- Express 4.x, Mongoose 8.x, Socket.IO 4.x
- JWT auth (jsonwebtoken + bcryptjs), express-validator, helmet, express-rate-limit, **cookie-parser**
- **web-push** — VAPID-based Web Push notifications (requires `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` env vars)
- **HttpOnly session cookie** — JWT issued as `HttpOnly; Secure; SameSite=Strict` cookie on login/register; `POST /api/auth/logout` clears it server-side

### Patterns — always follow these
```js
// Controller — always wrap with asyncHandler
const asyncHandler = require('../utils/asyncHandler');
exports.doThing = asyncHandler(async (req, res) => {
  res.json({ success: true, data: { ... } });
});

// Error — use AppError, never throw plain Error
const { AppError } = require('../utils/errors');
throw new AppError('ERROR_CODE', 'Human message', 404);

// Route — auth first, then validators, then controller
router.get('/path', auth, validators, controller.method);

// Transactions — wrap multi-doc writes
const withTransaction = require('../utils/withTransaction');
await withTransaction(async (session) => { ... });

// Audit — log all sensitive admin actions
const { logAdminAction } = require('../services/adminAuditService');
await logAdminAction({ action: 'invite.create', actorUserId, targetType: 'invite', targetId, after: doc });
```

### Response shape
```js
// Success
{ success: true, data: { ... } }

// Error
{ success: false, error: { code: 'ERROR_CODE', message: '...' } }
```

### Key files
| File | Purpose |
|------|---------|
| `src/app.js` | Express bootstrap, middleware, route mounting, request logging with token scrubbing |
| `src/config/db.js` | MongoDB connection with retry |
| `src/middleware/auth.js` | `auth`, `requireRole`, `requireOperationalRole` — reads token from `req.cookies.token` first, falls back to `Authorization: Bearer` header |
| `src/middleware/authorize.js` | `assertRoleEditAuthority`, `assertCompanyAccess`, `OPERATIONAL_ROLE_RANK`, `requireAdminOrCompanyCommander` |
| `src/middleware/errorHandler.js` | Global error handler |
| `src/utils/errors.js` | `AppError` class |
| `src/utils/asyncHandler.js` | Async wrapper for controllers |
| `src/utils/withTransaction.js` | MongoDB session/transaction helper |
| `src/utils/locationWriteBuffer.js` | `LocationWriteBuffer` — batches GPS location DB writes; `enqueue(userId, coords)` queues a write, `bulkWrite` fires every `LOCATION_WRITE_BUFFER_MS` ms (default 500); `shutdown()` drains on SIGTERM/SIGINT; `getStats()` for diagnostics |
| `src/utils/roles.js` | `OPERATIONAL_ROLES` array — source of truth for role enums |
| `src/utils/validators.js` | All express-validator chains: validateRegister (inviteToken required), validateCreateInvite, validateLogin, validateLocation, validateAOCreate/Update, validateFieldEventCreate, validateMobilizationCreate, validateMobilizationAdvance |
| `src/realtime/socket.js` | Socket.IO init, `getIO()` |
| `src/services/breachService.js` | Stateful AO breach detection |
| `src/services/locationService.js` | Location update handling; `updateUserLocation()` accepts optional `locationBuffer` — when provided, enqueues to `LocationWriteBuffer` instead of `User.save()` (broadcast still fires immediately); accepts optional `user` param to skip redundant `findById()` |
| `src/services/presenceService.js` | Connected user tracking |
| `src/services/scopeResolver.js` | Populates `req.scope` — what data the current user can see |
| `src/services/hierarchyService.js` | `resolveHierarchyPath()`, `ensureNoActiveChildren()`, `getHierarchyTree()` |
| `src/services/adminAuditService.js` | `logAdminAction()` — writes to AdminAuditLog collection |
| `src/controllers/inviteController.js` | `createInvite`, `validateInviteToken`, `listInvites`, `revokeInvite` |
| `src/controllers/eventController.js` | `createEvent`, `listEvents`, `acknowledgeEvent`, `resolveEvent` — fires push after create |
| `src/controllers/pushController.js` | `getVapidPublicKey`, `subscribe`, `unsubscribe` — Web Push subscription management |
| `src/models/InviteToken.js` | Invite token schema — token, roles, hierarchy, expiry, usedAt |
| `src/models/AdminAuditLog.js` | Audit trail schema — action, actorUserId, before/after |
| `src/models/FieldEvent.js` | Field event schema — eventType, senderId, GeoJSON Point coords, status lifecycle, denormalized hierarchy |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Moshe-I-Amar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
