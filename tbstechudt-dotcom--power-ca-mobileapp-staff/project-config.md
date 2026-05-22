---
trigger: always_on
description: **For GitHub Copilot, Codex, Cursor, and other AI coding assistants**
---

# AGENTS.md - AI Coding Assistant Guide

**For GitHub Copilot, Codex, Cursor, and other AI coding assistants**

This file contains critical rules and patterns for the PowerCA Mobile project. **READ THIS FIRST** before generating any code or suggesting changes.

---

## ✅ **CRITICAL FIX IMPLEMENTED!** ✅

**Sync engine is now SAFE for bidirectional sync with mobile data preservation!**

### What Was Fixed
The sync engine previously deleted mobile data with `TRUNCATE TABLE`. This is now **FIXED** with UPSERT logic.

### Changes
- ✅ Replaced `TRUNCATE + INSERT` with `INSERT ON CONFLICT DO UPDATE`
- ✅ Only updates `source='D'` records (desktop)
- ✅ Preserves `source='M'` records (mobile)
- ✅ Added incremental mode support

### NOW SAFE TO RUN
```bash
# ✅ Forward sync (preserves mobile data)
node sync/production/runner-staging.js --mode=full
node sync/production/runner-staging.js --mode=incremental

# ✅ Reverse sync (INSERT-only)
node sync/production/reverse-sync-engine.js

# ✅ Initialize metadata (run once)
node scripts/create-sync-metadata-table.js
```

**See:** [`docs/CRITICAL-STAGING-FLAW.md`](docs/CRITICAL-STAGING-FLAW.md) for complete fix details.

---

## 🚨 CRITICAL RULES - DO NOT VIOLATE

### RULE 1: NEVER Clear Production Tables Directly

**❌ FORBIDDEN PATTERNS:**
```javascript
// DO NOT USE THESE PATTERNS
await client.query('TRUNCATE TABLE jobshead');
await client.query('DELETE FROM jobshead');
// Then insert data...
```

**Why forbidden:** If sync fails mid-way, production data is LOST.

**✅ REQUIRED PATTERN: Use Staging Tables + UPSERT**
```javascript
// ALWAYS use this pattern for production syncs
await client.query('CREATE TEMP TABLE jobshead_staging (LIKE jobshead)');
// Load all data to staging
for (const record of records) {
  await client.query('INSERT INTO jobshead_staging VALUES (...)');
}
// Atomic UPSERT (preserves mobile data)
await client.query('BEGIN');
await client.query(`
  INSERT INTO jobshead SELECT * FROM jobshead_staging
  ON CONFLICT (job_id) DO UPDATE SET
    [columns] = EXCLUDED.[columns]
  WHERE jobshead.source = 'D' OR jobshead.source IS NULL
`);
await client.query('COMMIT');
// Mobile records (source='M') are PRESERVED! ✅
```

**Use these scripts ONLY:**
- `sync/runner-staging.js` ✅
- `sync/engine-staging.js` ✅

**NEVER use these scripts in production:**
- `sync/runner-optimized.js` ❌
- `sync/sync-missing-jobs.js` ❌
- Any script with `TRUNCATE` before loading staging ❌

---

### RULE 2: Skip Auto-Increment Columns in INSERT

**❌ WRONG:**
```javascript
INSERT INTO taskchecklist (tc_id, job_id, task_name)
VALUES (NULL, 123, 'Task 1')  // ❌ tc_id is auto-increment!
```

**✅ CORRECT:**
```javascript
// Skip tc_id - let PostgreSQL generate it
INSERT INTO taskchecklist (job_id, task_name)
VALUES (123, 'Task 1')  // ✅ tc_id auto-generated
```

**Auto-increment columns to skip:**
- `taskchecklist.tc_id`
- Any column with `DEFAULT nextval('...')`

**Code pattern:**
```javascript
const AUTO_INCREMENT_COLUMNS = ['tc_id'];
const columns = Object.keys(record).filter(col =>
  !AUTO_INCREMENT_COLUMNS.includes(col)
);
const query = `INSERT INTO table (${columns.join(',')}) VALUES (...)`;
```

---

### RULE 3: Handle Missing Foreign Keys

**Context:** Desktop PostgreSQL has NO FK constraints. Data contains:
- Orphaned records (jobs with deleted clients)
- Invalid references (client_id=500 doesn't exist)
- NULL values (con_id=0 or NULL)

**❌ DON'T assume FK constraints exist:**
```javascript
// This will fail on 16% of jobs
INSERT INTO jobshead (job_id, client_id) VALUES (1, 500);
// Error: violates foreign key constraint "jobshead_client_id_fkey"
```

**✅ DO one of these:**

**Option A: Remove FK constraint (chosen approach)**
```sql
ALTER TABLE jobshead DROP CONSTRAINT IF EXISTS jobshead_client_id_fkey;
```

**Option B: Pre-filter invalid records**
```javascript
const validClientIds = new Set(await getValidClientIds());
const validRecords = records.filter(r => validClientIds.has(r.client_id));
```

**Removed FK constraints:**
- `jobshead.client_id` → climaster
- `jobshead.con_id` → conmaster
- `jobtasks.task_id` → taskmaster
- `jobtasks.client_id` (made nullable)
- `taskchecklist.job_id` → jobshead
- `reminder.client_id` → climaster
- `remdetail.staff_id` → mbstaff

---

### RULE 4: Always Use Transactions for Multi-Step Operations

**❌ WRONG:**
```javascript
await client.query('DELETE FROM jobtasks');
await client.query('DELETE FROM jobshead');  // If this fails, jobtasks is empty!
```

**✅ CORRECT:**
```javascript
await client.query('BEGIN');
try {
  await client.query('DELETE FROM jobtasks');
  await client.query('DELETE FROM jobshead');
  await client.query('COMMIT');
} catch (error) {
  await client.query('ROLLBACK');  // Restore both tables
  throw error;
}
```

---

### RULE 5: Respect FK Dependency Order

**When deleting/truncating tables with FK relationships:**

**❌ WRONG ORDER:**
```javascript
await client.query('TRUNCATE jobshead');  // ❌ Fails! jobtasks references it
await client.query('TRUNCATE jobtasks');
```

**✅ CORRECT ORDER (child before parent):**
```javascript
await client.query('TRUNCATE jobtasks');   // Child first
await client.query('TRUNCATE jobshead');   // Parent second
```

**Dependency hierarchy:**
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tbstechudt-dotcom/Power-CA_MobileApp_Staff](https://github.com/tbstechudt-dotcom/Power-CA_MobileApp_Staff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
