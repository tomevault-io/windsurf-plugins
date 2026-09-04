---
trigger: always_on
description: Dutch rural tourism voucher web app — consumer browse/claim + business management + admin approval.
---

# Plattelandsbon (Oom Gerrit)

Dutch rural tourism voucher web app — consumer browse/claim + business management + admin approval.
Part of the "Oom Gerrit" brand. This is a standalone git repo at `~/Dev/oomgerrit/plattelandsbon/`.

## Architecture: Onion Architecture (Strict)

ALL code follows onion architecture with strict modularity. Dependencies ALWAYS point inward.

```
Presentation (app/, components/) → Server (tRPC routers) → Application (use cases) → Domain (entities)
                                                                    ↑
                                          Infrastructure (Prisma repos, AI, email, scraping)
```

### Layers

**Domain** (`src/domain/`) — ZERO external dependencies
- `entities/` — Business objects with protected constructors + `static create()`/`fromProps()` factories
- `value-objects/` — Immutable typed values (ClaimCode, VoucherStatus, DiscountType)
- `repositories/` — INTERFACES only (IVoucherRepository, IClaimRepository) — all methods accept optional `tx?: TransactionContext`
- `types/` — TransactionContext (opaque), PaginationOptions, ITransactionManager
- `services/` — Pure domain logic (VoucherValidationService, FraudDetectionService)
- `errors/` — Domain-specific errors

**Application** (`src/application/`) — Orchestrates domain, no framework deps
- `interfaces/` — `IUseCase<TInput, TOutput>` generic interface
- `use-cases/` — One class per operation, all implement `IUseCase`
- `dtos/` — Data transfer objects for layer boundaries
- `mappers/` — Entity <-> DTO conversions

**Infrastructure** (`src/infrastructure/`) — Framework/external deps live here
- `repositories/` — Prisma implementations extending `BaseRepository<TEntity, TPrismaRecord>`
- `services/PrismaTransactionManager.ts` — implements `ITransactionManager` for cross-repo transactions
- `services/ai/` — Anthropic API integration
- `services/email/` — Resend + react-email templates
- `services/scraping/` — Competitor scrapers (Groupon, SocialDeal, WeekendjeWeg)
- `services/storage/` — Vercel Blob
- `services/cache/` — Upstash Redis
- `config/container.ts` — Dependency wiring (manual factory functions)

**Server** (`src/server/api/`) — Thin tRPC layer
- Routers call use cases, nothing else
- One router per domain (vouchers, claims, businesses, categories, users, auth)

**Presentation** (`src/app/`, `src/components/`)
- Next.js App Router pages — render only
- React components — no business logic

### Key Patterns

**Entity creation:**
```typescript
// New entity (use cases/domain services):
const business = Business.create(props)
// Reconstitution from DB (infrastructure repos):
const business = Business.fromProps(props)
// Direct `new Entity()` is forbidden (protected constructor)
```

**Use case signature:**
```typescript
class RegisterBusinessUseCase implements IUseCase<BusinessRegistrationDTO, BusinessResponseDTO> {
  constructor(private businessRepo: IBusinessRepository) {}
  async execute(input: BusinessRegistrationDTO): Promise<BusinessResponseDTO> { ... }
}
```

**Repository with transactions:**
```typescript
// All repo methods accept optional tx:
findById(id: string, tx?: TransactionContext): Promise<Entity | null>
// Transaction usage:
await transactionManager.run(async (tx) => {
  await voucherRepo.incrementClaimCount(voucherId, tx)
  await claimRepo.create(claimData, tx)
})
```

**BaseRepository (infrastructure):**
```typescript
class PrismaBusinessRepo extends BaseRepository<Business, PrismaRecord> implements IBusinessRepository {
  protected toDomain(record: PrismaRecord): Business { return Business.fromProps(...) }
  // Uses this.getClient(tx), this.mapOrNull(), this.mapMany()
}
```

### Rules
- Domain layer imports NOTHING from other layers
- Entities have protected constructors — use `create()` or `fromProps()`
- All use cases implement `IUseCase<TInput, TOutput>`
- All Prisma repos extend `BaseRepository` and accept `tx?: TransactionContext`
- tRPC routers are thin: validate input, call use case, return result
- Prisma is NEVER imported outside `src/infrastructure/`
- Use cases depend on repository interfaces, not implementations
- Components never call Prisma or use cases directly

## Tech Stack

Next.js 16, React 19, TypeScript 5.9, PostgreSQL, Prisma 6, NextAuth v5, tRPC v11, Tailwind CSS 4, shadcn/ui, GSAP + @gsap/react (hero scroll animations), Anthropic AI SDK, Resend, Vercel Blob, Upstash Redis

See `RESEARCH.md` for validated versions and configuration details.

## File Organization

- tRPC routers → `src/server/api/routers/`, one per domain
- Use cases → `src/application/use-cases/`, grouped by domain
- Use case interface → `src/application/interfaces/IUseCase.ts`
- Domain entities → `src/domain/entities/` (Business, Voucher, VoucherClaim, Category, User)
- Domain types → `src/domain/types/` (TransactionContext, PaginationOptions)
- Value objects → `src/domain/value-objects/` (ClaimStatus, VoucherStatus, DiscountType, UserRole, BusinessStatus)
- Repository interfaces → `src/domain/repositories/` (IVoucherRepository, IClaimRepository, IBusinessRepository, IUserRepository, ICategoryRepository)
- Repository implementations → `src/infrastructure/repositories/`
- Base repository → `src/infrastructure/repositories/BaseRepository.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willem4130/plattelandsbon](https://github.com/willem4130/plattelandsbon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
