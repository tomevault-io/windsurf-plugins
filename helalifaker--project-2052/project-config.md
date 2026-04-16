---
trigger: always_on
description: - Financial planning tool for 30-year projections (2023-2053)
---

# Project Zeta / Project_2052 - Cursor Rules
## Financial Planning Application

**Project Context:**
- Financial planning tool for 30-year projections (2023-2053)
- Millions of SAR in calculations
- Board-level decision support
- Three distinct calculation periods (Historical, Transition, Dynamic)
- Circular financial dependencies

**Tech Stack:**
- Next.js 16 (App Router)
- React 19
- TypeScript 5.7 (strict mode required)
- Prisma (PostgreSQL)
- Decimal.js (for all financial calculations)
- Zod (runtime validation)
- Vitest (testing)

---

## CRITICAL RULES - NEVER VIOLATE

### 1. Financial Calculations - ZERO TOLERANCE

**ALWAYS use Decimal.js for money. NEVER use JavaScript numbers.**

```typescript
// ✅ CORRECT
import Decimal from 'decimal.js';
const revenue = new Decimal('125300000');
const rent = revenue.times(0.08);
const netIncome = revenue.minus(rent);

// ❌ WRONG - Will cause precision errors
const revenue = 125300000;
const rent = revenue * 0.08; // Precision loss!
```

**Pre-create Decimal constants (performance critical):**
```typescript
// ✅ CORRECT - Pre-create constants
export const DECIMAL_ZERO = new Decimal(0);
export const DECIMAL_ONE = new Decimal(1);
export const ZAKAT_RATE = new Decimal(0.025);

// ❌ WRONG - Don't recreate in loops
function calculateZakat(ebt: Decimal): Decimal {
  const zakatRate = new Decimal(0.025); // Created every call - wasteful!
  return ebt.times(zakatRate);
}
```

**Compare Decimals using methods, not operators:**
```typescript
// ✅ CORRECT
if (cash.greaterThanOrEqualTo(minCash)) { ... }
if (ebt.lessThanOrEqualTo(DECIMAL_ZERO)) { ... }

// ❌ WRONG
if (cash >= minCash) { ... } // Type error + precision loss
if (ebt.toNumber() <= 0) { ... } // Loses precision
```

### 2. Type Safety - MANDATORY

**Never use `any` type. Always type explicitly.**

```typescript
// ✅ CORRECT
function calculateRent(
  baseRent: Decimal,
  growthRate: number,
  years: number
): Decimal {
  return baseRent.times(Math.pow(1 + growthRate, years));
}

// ❌ WRONG
function calculateRent(baseRent: any, growthRate: any): any {
  return baseRent * (1 + growthRate);
}
```

**Use Zod for runtime validation:**
```typescript
// ✅ CORRECT
import { z } from 'zod';
const ProposalSchema = z.object({
  name: z.string().min(1),
  baseRent2028: z.number().positive(),
  rentModel: z.enum(['FIXED', 'REVSHARE', 'PARTNER'])
});

// ❌ WRONG
const proposal = await request.json(); // No validation!
```

### 3. API Routes - Security & Validation Required

**Every API route MUST:**
1. Authenticate and authorize (RBAC)
2. Validate input with Zod
3. Handle errors properly
4. Return appropriate status codes

```typescript
// ✅ CORRECT
export async function POST(request: NextRequest) {
  try {
    // 1. Authenticate & authorize
    const user = await requireAuth(request, [Role.ADMIN, Role.PLANNER]);
    
    // 2. Parse & validate
    const body = await request.json();
    const validated = CreateProposalSchema.parse(body);
    
    // 3. Business logic
    const proposal = await db.leaseProposal.create({
      data: { ...validated, createdBy: user.id }
    });
    
    return NextResponse.json(proposal, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// ❌ WRONG - No auth, no validation, no error handling
export async function POST(request: NextRequest) {
  const body = await request.json();
  const proposal = await db.leaseProposal.create({ data: body });
  return NextResponse.json(proposal);
}
```

### 4. React & Next.js - Performance & Best Practices

**Server Components by default. 'use client' only when needed.**

```typescript
// ✅ CORRECT - Server Component (default)
// app/proposals/[id]/page.tsx
export default async function ProposalPage({ params }: { params: { id: string } }) {
  const proposal = await db.leaseProposal.findUnique({
    where: { id: params.id }
  });
  return <ProposalDetail proposal={proposal} />;
}

// ✅ CORRECT - Client Component (only when needed)
// components/ProposalDetail.tsx
'use client';
export function ProposalDetail({ proposal }: { proposal: Proposal }) {
  const [activeTab, setActiveTab] = useState('overview');
  return <Tabs value={activeTab} onValueChange={setActiveTab}>...</Tabs>;
}
```

**Memoize expensive computations:**
```typescript
// ✅ CORRECT
const FinancialTable = memo(({ data }: { data: FinancialPeriod[] }) => {
  const formattedData = useMemo(() => {
    return data.map(period => ({
      year: period.year,
      revenue: formatMillions(period.revenue),
      rent: formatMillions(period.rent)
    }));
  }, [data]);
  return <Table data={formattedData} />;
});

// ❌ WRONG - Recalculates every render
function FinancialTable({ data }: { data: FinancialPeriod[] }) {
  const formattedData = data.map(period => formatMillions(period.revenue));
  return <Table data={formattedData} />;
}
```

**Debounce interactive inputs (300ms):**
```typescript
// ✅ CORRECT
import { useDebouncedCallback } from 'use-debounce';
const debouncedRecalculate = useDebouncedCallback(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/helalifaker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
