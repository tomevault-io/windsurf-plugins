---
trigger: always_on
description: > **Doc map:** [`MASTER.md`](MASTER.md) is the canonical product spec (mission, ethics, rating instrument, privacy, schema). [`ARCHITECTURE.md`](ARCHITECTURE.md) is the technical architecture reference. [`brand.md`](brand.md) is the voice and visual brand bible. [`CLAUDE_CONTEXT.md`](CLAUDE_CONTEXT.md) is the orientation doc for Claude Code agents. This file is coding conventions only.
---

# RateMyPlace Boston - Coding Conventions

> **Doc map:** [`MASTER.md`](MASTER.md) is the canonical product spec (mission, ethics, rating instrument, privacy, schema). [`ARCHITECTURE.md`](ARCHITECTURE.md) is the technical architecture reference. [`brand.md`](brand.md) is the voice and visual brand bible. [`CLAUDE_CONTEXT.md`](CLAUDE_CONTEXT.md) is the orientation doc for Claude Code agents. This file is coding conventions only.

## Quick Reference

| Item | Convention |
|------|------------|
| Framework | Astro 5.x SSR + React islands |
| Database | Cloudflare D1 (SQLite) |
| Auth | Lucia v3 with D1 adapter |
| Styling | Tailwind CSS 4.x |
| Types | TypeScript strict mode |

## File Patterns

### Pages (Astro)
- **Public pages**: `src/pages/*.astro` - SSR, no client JS unless needed
- **Dynamic routes**: `src/pages/[slug].astro` - Use `Astro.params.slug`
- **Admin pages**: `src/pages/admin/*.astro` - Always check `locals.user?.isAdmin`

### API Routes
- **Location**: `src/pages/api/**/*.ts`
- **Auth check pattern**:
```typescript
if (!context.locals.user) {
  return new Response(JSON.stringify({ error: 'Authentication required' }), {
    status: 401,
    headers: { 'Content-Type': 'application/json' }
  });
}
```
- **Admin check pattern**:
```typescript
if (!context.locals.user?.isAdmin) {
  return new Response(JSON.stringify({ error: 'Admin access required' }), {
    status: 403,
    headers: { 'Content-Type': 'application/json' }
  });
}
```

### Components
- **Astro components**: Static/SSR rendering, use for layouts and data display
- **React components**: Interactive islands only (forms, maps, dynamic UI)
- **React directive**: Always use `client:load` for immediate interactivity

```astro
<!-- Astro component with React island -->
<ReviewForm client:load buildingId={building.id} />
```

### Library Files (`src/lib/`)
- **Single responsibility**: One concern per file
- **Export types**: Always export interfaces for consumers
- **Critical files**:
  - `scoring.ts` - All scoring logic (weights, calculations)
  - `surveyItems.ts` - Survey questions and help text
  - `validation.ts` - Input validation
  - `audit.ts` - Admin action logging

## Database Patterns

### Getting DB Connection
```typescript
import { getDB } from '../../lib/db';

const db = getDB((context.locals as any).runtime);
```

### Query Patterns
```typescript
// Single row
const user = await db.prepare('SELECT * FROM users WHERE id = ?')
  .bind(userId)
  .first<User>();

// Multiple rows
const { results } = await db.prepare('SELECT * FROM reviews WHERE building_id = ?')
  .bind(buildingId)
  .all<Review>();

// Insert with generated ID
import { generateIdFromEntropySize } from 'lucia';
const id = generateIdFromEntropySize(10);
await db.prepare('INSERT INTO reviews (id, ...) VALUES (?, ...)')
  .bind(id, ...)
  .run();
```

### Timestamps
- Use `unixepoch()` for SQLite timestamps (not `datetime('now')`)
- Column type: `INTEGER DEFAULT (unixepoch())`

## Scoring System (Critical)

### Modifying Weights
1. Edit `ITEM_WEIGHTS` in `src/lib/scoring.ts`
2. Document justification with academic citation
3. Update `src/pages/methodology.astro`

### Adding Survey Items
1. Add column in new migration (`migrations/XXXX_name.sql`)
2. Add to `src/lib/surveyItems.ts` with help text
3. Add to domain array in `src/lib/scoring.ts` (UNIT_FIELDS, BUILDING_FIELDS, or LANDLORD_FIELDS)
4. Set weight in `ITEM_WEIGHTS`
5. Update `ReviewForm.tsx` and `ReviewCard.astro`

### Weight Guidelines
| Weight | Use For |
|--------|---------|
| 1.5x | Major health hazards (pests, mold) |
| 1.3x | Safety hazards (structural, climate) |
| 1.2x | Health-adjacent (plumbing, security) |
| 1.0x | Standard quality factors |

## Error Handling

### API Responses
```typescript
// Success
return new Response(JSON.stringify({ data: result }), {
  status: 200,
  headers: { 'Content-Type': 'application/json' }
});

// Client error
return new Response(JSON.stringify({ error: 'Validation failed', details: errors }), {
  status: 400,
  headers: { 'Content-Type': 'application/json' }
});

// Server error
return new Response(JSON.stringify({ error: 'Internal server error' }), {
  status: 500,
  headers: { 'Content-Type': 'application/json' }
});
```

### Audit Logging (Admin Actions)
```typescript
import { createAuditLog } from '../../lib/audit';

// Best-effort logging - failures don't break the action
await createAuditLog(db, {
  adminUserId: context.locals.user.id,
  actionType: 'review_approved',
  entityType: 'review',
  entityId: reviewId,
  oldValue: { status: 'pending' },
  newValue: { status: 'approved' }
});
```

## Styling

### Score Colors

**Single source of truth: [`src/lib/scoring-colors.ts`](src/lib/scoring-colors.ts).** Always import from there. Do not roll your own thresholds or labels.

Canonical four-band system (mirrors `brand.md` §4.2):

| Band | Range | Label | Fill | Text | Hex |
|------|-------|-------|------|------|-----|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mereditharmcgee/ratemyplace](https://github.com/mereditharmcgee/ratemyplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
