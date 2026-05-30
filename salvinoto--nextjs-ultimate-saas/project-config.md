---
trigger: always_on
description: Next.js Ultimate SaaS template with Better Auth, Polar payments, and usage-based metering
---


# Next.js Ultimate SaaS Template

This project is a full-featured SaaS starter with authentication, payments, organizations, and usage-based billing.

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Database**: PostgreSQL with Prisma ORM
- **Auth**: Better Auth with organization support
- **Payments**: Polar for subscriptions and usage-based billing
- **Styling**: Tailwind CSS + shadcn/ui components
- **Email**: Resend

## Project Structure

```
lib/
├── auth.ts              # Better Auth configuration
├── auth-client.ts       # Client-side auth hooks
├── payments.ts          # Polar payments integration
├── metering/
│   ├── index.ts         # Unified metering exports
│   ├── client.ts        # Usage tracking functions
│   ├── limits.ts        # Limit checking functions
│   ├── types.ts         # TypeScript types
│   └── setup-meters.ts  # Meter setup script
├── permissions.ts       # RBAC roles and permissions
└── plans/
    └── db/
        └── customer.ts  # Customer database operations
```

---

## Authentication (Better Auth)

### Getting the Current Customer

Always use `getCurrentCustomer()` in server actions and API routes:

```typescript
import { getCurrentCustomer } from '@/lib/payments';

const { user, organization, billingEntityId } = await getCurrentCustomer();
```

- `user` - The authenticated user from session
- `organization` - The active organization (if any)
- `billingEntityId` - Use this for metering (automatically picks org or user ID)

### Getting Active Subscription

```typescript
import { getActiveSubscription } from '@/lib/payments';

const subscription = await getActiveSubscription();
```

---

## RBAC Permissions

Roles are defined in `lib/permissions.ts`:

| Role | Permissions |
|------|-------------|
| `owner` | Full control: org update/delete, member CRUD, invitations, project CRUD |
| `admin` | Org update, member CRUD, invitations, project create/update |
| `member` | Project create only |

---

## Usage-Based Billing (Metering)

### Available Meters

| Slug | Event Name | Aggregation | Use Case |
|------|------------|-------------|----------|
| `api_requests` | `api.request` | count | API calls per billing period |
| `storage_gb` | `storage.update` | max(size_gb) | Peak storage used |
| `ai_tokens` | `ai.tokens` | sum(tokens) | Total AI tokens consumed |
| `team_seats` | `seat.active` | unique(user_id) | Active team members |

### Tracking Usage

Always import from `@/lib/metering`:

```typescript
import { 
  trackApiRequest,
  trackAiTokens,
  trackStorageUpdate,
  trackSeatActivity,
  trackUsage,
  trackUsageBatch,
} from '@/lib/metering';
```

#### Track API Requests

```typescript
const { billingEntityId } = await getCurrentCustomer();
await trackApiRequest(billingEntityId, '/api/endpoint');

// With options
await trackApiRequest(billingEntityId, '/api/endpoint', {
  method: 'POST',
  statusCode: 200,
  duration: 150
});
```

#### Track AI Tokens

```typescript
await trackAiTokens(billingEntityId, 1500);
await trackAiTokens(billingEntityId, 1500, { model: 'gpt-4', type: 'output' });
```

#### Track Storage

```typescript
await trackStorageUpdate(billingEntityId, 2.5); // 2.5 GB
await trackStorageUpdate(billingEntityId, 3.0, 'upload');
```

#### Track Seat Activity

```typescript
await trackSeatActivity(orgId, userId);
await trackSeatActivity(orgId, userId, 'login');
```

#### Batch Tracking

```typescript
await trackUsageBatch([
  { externalCustomerId: id, eventName: 'api.request', properties: { endpoint: '/a' } },
  { externalCustomerId: id, eventName: 'api.request', properties: { endpoint: '/b' } },
]);
```

### Checking Limits

```typescript
import { 
  checkCurrentLimit,
  checkLimit,
  getAllUsage,
} from '@/lib/metering';

// Current customer's limit
const status = await checkCurrentLimit('api_requests');
// { allowed: boolean, current: number, limit: number | null, remaining: number | null }

// Specific customer
const status = await checkLimit('user_123', 'storage_gb');

// All meters
const allUsage = await getAllUsage();
```

### Protecting Actions with Limits

#### Using withUsageLimit (throws on limit exceeded)

```typescript
import { withUsageLimit, trackAiTokens } from '@/lib/metering';

export async function generateContent(prompt: string) {
  const { billingEntityId } = await getCurrentCustomer();
  
  return withUsageLimit('ai_tokens', async () => {
    const result = await callAI(prompt);
    await trackAiTokens(billingEntityId, result.tokensUsed);
    return result;
  });
}
```

#### Using withUsageLimitSafe (returns result object)

```typescript
import { withUsageLimitSafe } from '@/lib/metering';

const result = await withUsageLimitSafe('storage_gb', async () => {
  return await uploadFile(file);
});

if (result.success) {
  console.log(result.data);
} else {
  console.error(result.error); // "Usage limit exceeded..."
}
```

### In API Routes

```typescript
export async function POST(req: Request) {
  const { billingEntityId } = await getCurrentCustomer();
  
  // Check limit first
  const status = await checkCurrentLimit('api_requests');
  if (!status.allowed) {
    return Response.json({ error: status.reason }, { status: 429 });
  }
  
  // Process request...
  
  // Track usage AFTER success

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salvinoto/nextjs-ultimate-saas](https://github.com/salvinoto/nextjs-ultimate-saas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
