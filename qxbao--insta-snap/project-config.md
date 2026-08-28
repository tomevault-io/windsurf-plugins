---
trigger: always_on
description: InstaSnap is a browser extension for tracking Instagram follower/following lists using **differential storage** architecture. It takes periodic snapshots and efficiently stores only deltas between snapshots (every 20th is a full checkpoint).
---

# InstaSnap AI Coding Guide

## Project Overview
InstaSnap is a browser extension for tracking Instagram follower/following lists using **differential storage** architecture. It takes periodic snapshots and efficiently stores only deltas between snapshots (every 20th is a full checkpoint).

**Tech Stack**: Vue 3 + TypeScript, Vite + @crxjs/vite-plugin, Pinia stores, Vitest, Tailwind CSS 4, IndexedDB (Dexie), pnpm

## Architecture & Key Patterns

### Multi-Context Browser Extension
The extension has 4 execution contexts that communicate via `browser.runtime.sendMessage`:
1. **Background Service Worker** ([src/background.ts](../src/background.ts)) - Orchestrates lifecycle events
2. **Background Service** ([src/utils/bg-service.ts](../src/utils/bg-service.ts)) - Core business logic, encryption, alarms, message routing
3. **Content Script** ([src/content/content.ts](../src/content/content.ts)) - Injected into Instagram pages, scrapes data via Instagram GraphQL
4. **Popup** ([src/popup/Popup.vue](../src/popup/Popup.vue)) - Quick snapshot actions on current profile
5. **Dashboard** ([src/dashboard/Dashboard.vue](../src/dashboard/Dashboard.vue)) - Full-page analytics UI

### Security Architecture (NEW)
**End-to-End Encryption** for sensitive credentials:
- All Instagram API tokens (`appId`, `csrfToken`, `wwwClaim`) are **encrypted at rest**
- AES-GCM encryption via Web Crypto API in [src/utils/encrypt.ts](../src/utils/encrypt.ts)
- Master encryption key stored as `CryptoKey` in IndexedDB `internalConfig` table
- `BackgroundService` manages encryption lifecycle:
  - `checkSecurityConfig()` - Initialize/load encryption key on startup
  - `ensureReady()` - Guard for secure operations
  - Auto-migration from legacy `browser.storage.local` to encrypted IndexedDB

**Encryption Flow**:
```typescript
// Write encrypted config
await database.writeEncryptedConfig("appId", value, encryptor);

// Read encrypted config  
const appId = await database.readEncryptedConfig("appId", encryptor);
```

### Storage Architecture (CRITICAL)
**IndexedDB with Dexie.js** in [src/utils/database.ts](../src/utils/database.ts):
- Every **20 snapshots** creates a **checkpoint** (full follower/following lists)
- Intermediate snapshots store only **deltas** (`add`/`rem` arrays) - **~95% storage reduction**
- Four main tables: `userMetadata`, `snapshots`, `crons`, `internalConfig` (NEW)
- Compound indexes for efficient queries: `[belongToId+timestamp]`, `[belongToId+isCheckpoint+timestamp]`
- **Always** use `database.saveSnapshot()` - it handles checkpoint vs delta logic automatically

**Database Tables Schema**:
1. **userMetadata**: Basic user info (id, username, fullName, avatarURL, updatedAt) - updated on every snapshot
2. **snapshots**: Differential storage with `belongToId`, `isCheckpoint` (1=full, 0=delta), `timestamp`, `followers/following` (add/rem arrays)
3. **crons**: Scheduled snapshots with `uid`, `interval` (1-168 hours), `lastRun` timestamp
4. **internalConfig**: Encrypted credentials stored as `{encrypted: ArrayBuffer, iv: Uint8Array}` (EncryptedData struct)

**Key Database Methods**:
- `saveSnapshot(uid, timestamp, followerIds, followingIds)` - Auto checkpointing
- `getFullList(uid, upToTimestamp?, isFollowers)` - Rebuild full list from checkpoint + deltas
- `getSnapshotHistory(uid, isFollowers)` - Get history timeline
- `getAllTrackedUsersWithMetadata()` - Dashboard data
- `saveCron(uid, interval, lastRun)` / `getCron(uid)` / `deleteCron(uid)` - Cron management
- `writeEncryptedConfig(key, value, encryptor)` / `readEncryptedConfig(key, encryptor)` - Secure storage (NEW)
- `bulkUpsertUserMetadata(users[])` - Efficient batch updates for user metadata

**Storage Migration Path**:
1. Legacy: `browser.storage.local` (plaintext credentials) ❌
2. Current: IndexedDB `internalConfig` (encrypted) ✅
3. Auto-migration runs on extension install/update

browser.storage is **only used for**:
- `browser.storage.session` for user locks (prevent duplicate snapshots)
- ⚠️ **Do NOT use `browser.storage.local` for new features** - use encrypted IndexedDB

### Instagram Data Collection
[src/utils/instagram.ts](../src/utils/instagram.ts) uses Instagram's private GraphQL API:
- Query hashes: `c76146de99bb02f6415203be841dd25a` (followers), `d04b0a864b4b54837c0d870b0e77e076` (following)
- Paginated requests (50 users/request) using `after` cursors
- Requires `appId`, `csrfToken`, `wwwClaim` extracted from page HTML
- CORS handled by declarativeNetRequest rules ([src/constants/rules.ts](../src/constants/rules.ts))

**Rate Limiting & Retry Logic**:
- Instagram API: **50 users per request** (MAX_USERS_PER_REQUEST constant)
- **IGFetch defaults**: maxRetries: 3, retryDelay: 2000ms, timeout: 15000ms
- Exponential backoff with jitter between cron snapshots (5-15s delay)
- Respects `Retry-After` header for 429 (rate limit) responses
- Max retry delay capped at **1 minute** for rate limit responses
- Cron snapshots process users sequentially to avoid rate limiting

### State Management
Three Pinia stores ([src/stores/](../src/stores/)):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qxbao/insta-snap](https://github.com/qxbao/insta-snap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
