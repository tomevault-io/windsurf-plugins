---
trigger: always_on
description: MicroCommerce is a showcase e-commerce platform demonstrating modern .NET microservices architecture. Users can browse products, add to cart, and checkout through a Next.js storefront. The backend is a modular monolith designed for gradual extraction to microservices.
---

# CLAUDE.md

## Project Overview

MicroCommerce is a showcase e-commerce platform demonstrating modern .NET microservices architecture. Users can browse products, add to cart, and checkout through a Next.js storefront. The backend is a modular monolith designed for gradual extraction to microservices.

## Tech Stack

- **Backend:** .NET 10, ASP.NET Core Minimal APIs, .NET Aspire 13.1.0
- **Gateway:** YARP reverse proxy (CORS, rate limiting, auth, routing)
- **Frontend:** Next.js 16, React 19, TypeScript 5, Tailwind CSS v4, TanStack React Query v5, Radix UI
- **Auth:** Keycloak (backend JWT + NextAuth.js v5 frontend)
- **Database:** PostgreSQL (schema-per-feature, shared database `appdb`)
- **Messaging:** Azure Service Bus (MassTransit 9.0) with emulator for dev; RabbitMQ for Kubernetes deployments
- **Patterns:** CQRS (MediatR), DDD, Event-Driven, Vertical Slice Architecture
- **Key Libraries:** Vogen (strongly typed IDs), Ardalis.SmartEnum, Ardalis.Specification, Ardalis.GuardClauses, FluentValidation, FluentResults

## UI Design Files (`design/`)

Design files are split by page for reduced AI context. Each file is self-contained with its referenced components + shared variables.

| File | Contents |
|---|---|
| `design/design-system.pen` | All 26 reusable components (buttons, badges, inputs, cards, nav items, etc.) |
| `design/storefront-homepage.pen` | Homepage screen (hero, categories, featured products, footer) |
| `design/storefront-catalog.pen` | Product catalog with filters and grid |
| `design/storefront-product-detail.pen` | Product detail page with tabs |
| `design/storefront-cart.pen` | Shopping cart page |
| `design/storefront-checkout.pen` | Checkout flow |
| `design/storefront-order-confirmation.pen` | Order confirmation page |
| `design/storefront-order-history.pen` | Order history list |
| `design/storefront-profile.pen` | User profile page |
| `design/auth-login.pen` | Login/auth page |
| `design/admin-dashboard.pen` | Admin dashboard with sidebar |

Legacy monolithic file: `pencil-new.pen` (kept as reference)

### Design → Code Workflow

```
"Generate React code from design/storefront-homepage.pen"
"Create a TypeScript component from design/design-system.pen"
"Export design/storefront-cart.pen as a Next.js page component"
"Generate code using Shadcn UI components from design/storefront-checkout.pen"
```

## Project Structure

```
src/
  MicroCommerce.AppHost/              # Aspire orchestrator (entry point)
  MicroCommerce.ApiService/           # Backend API
    Features/                         # Vertical slices
      Catalog/                        # Products, categories, images
      Cart/                           # Shopping cart (guest + auth)
      Ordering/                       # Checkout, orders, saga
      Inventory/                      # Stock management
      Profiles/                       # User profiles, addresses, avatars
      Reviews/                        # Product reviews (verified purchase)
      Wishlists/                      # Authenticated user wishlists
      Messaging/                      # Dead letter queue UI
    Common/                           # Shared infrastructure
      Behaviors/                      # MediatR pipeline (validation, result validation)
      Messaging/                      # DeadLetterQueueService, DomainEventFaultConsumer
      Persistence/                    # BaseDbContext, interceptors, conventions, OutboxDbContext
      Exceptions/                     # Global exception handling
      Extensions/                     # FluentResults -> HTTP mapping
      OpenApi/                        # Vogen/SmartEnum schema transformers
  MicroCommerce.Gateway/              # YARP reverse proxy
  MicroCommerce.ServiceDefaults/      # Aspire cross-cutting (telemetry, health)
  MicroCommerce.ApiService.Tests/     # xUnit integration + unit tests
    Integration/                      # Testcontainers + WebApplicationFactory
    Unit/                             # Domain logic unit tests
  MicroCommerce.Web/                  # Next.js frontend
    src/app/(storefront)/             # Customer-facing routes
    src/app/admin/                    # Admin dashboard
    src/components/                   # React components
    src/hooks/                        # Custom hooks (TanStack Query)
    src/lib/                          # API client, utilities
    src/types/                        # Type augmentations (next-auth.d.ts)
    src/auth.ts                       # NextAuth.js configuration
    src/middleware.ts                  # Next.js middleware (auth route protection)
    e2e/                              # Playwright E2E tests
  BuildingBlocks/BuildingBlocks.Common/ # DDD primitives (aggregates, events, value objects)
```

## Build & Run

```bash
# Run the full stack via Aspire
dotnet run --project src/MicroCommerce.AppHost

# Frontend only
cd src/MicroCommerce.Web && npm install && npm run dev

# Backend only
dotnet run --project src/MicroCommerce.ApiService

# Run backend tests
dotnet test src/MicroCommerce.ApiService.Tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baotoq/micro-commerce](https://github.com/baotoq/micro-commerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
