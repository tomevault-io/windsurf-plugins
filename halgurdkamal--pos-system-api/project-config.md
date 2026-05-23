---
trigger: always_on
description: **Multi-Tenant Pharmacy POS System** - A centralized Point-of-Sale API for pharmaceutical retail chains.
---

# POS System API - AI Coding Agent Instructions

## Product Overview

**Multi-Tenant Pharmacy POS System** - A centralized Point-of-Sale API for pharmaceutical retail chains.

### Core Concept

This system implements a **shared database architecture** where:

- **Centralized Drug Database**: One master database contains all drug information (barcodes, formulations, regulatory info, pricing guidelines)
- **Multi-Shop Registration**: Multiple pharmacy shops can register and use the system
- **Shop-Specific Inventory**: Each shop maintains its own stock levels, batches, and local pricing while referencing the central drug catalog
- **Shared Drug Catalog**: All shops access the same verified drug information (brand names, generic names, manufacturers, barcodes)

### Key Features

1. **Central Drug Repository**: Master drug catalog with complete pharmaceutical data (formulations, regulations, suppliers)
2. **Shop Registration System**: Pharmacies register and get isolated inventory management
3. **Per-Shop Inventory**: Each shop tracks its own:
   - Stock levels (quantity on hand)
   - Batch numbers and expiry dates
   - Purchase prices and selling prices
   - Storage locations
4. **Barcode Management**: Universal barcode system across all registered shops
5. **Multi-Tenant Data Isolation**: Shop-specific data segregation while sharing drug catalog

### Business Model

- **For Pharmacy Chains**: Corporate chains can manage multiple locations with consistent drug data
- **For Independent Pharmacies**: Individual shops benefit from centralized drug database without managing it
- **For Distributors**: Can provide verified drug information to multiple retail partners

## Architecture Overview

This is a **Clean Architecture ASP.NET Core 6.0 API** implementing the multi-tenant POS system. The codebase follows strict separation of concerns with dependency flow always pointing inward (API → Application → Domain).

### Layer Structure

```
src/
├── Core/Domain/              # Pure business entities (NO dependencies)
│   ├── Common/BaseEntity.cs  # All entities inherit from this
│   └── {Feature}/            # Feature-based organization
│       ├── Entities/
│       │   ├── Drug.cs       # Shared drug catalog (all shops)
│       │   ├── Shop.cs       # Shop registration & profile
│       │   ├── ShopInventory.cs  # Per-shop stock levels
│       │   └── Order.cs      # Shop-specific transactions
│       └── ValueObjects/
│           ├── Pricing.cs    # Base pricing + shop-specific pricing
│           ├── Formulation.cs # Drug formulation details
│           ├── Batch.cs      # Shop-specific batch tracking
│           └── Inventory.cs  # Stock management per shop
├── Core/Application/         # Use cases via CQRS (depends ONLY on Domain)
│   ├── Common/
│   │   ├── Interfaces/       # Repository contracts
│   │   │   ├── IDrugRepository.cs       # Central drug catalog
│   │   │   ├── IShopRepository.cs       # Shop management
│   │   │   └── IInventoryRepository.cs  # Per-shop inventory
│   │   └── Models/           # Result<T>, PagedResult<T>
│   └── {Feature}/
│       ├── Queries/          # Read operations (GetDrug, GetShopInventory)
│       ├── Commands/         # Write operations (RegisterShop, UpdateStock)
│       └── DTOs/             # API response models
├── Infrastructure/           # External concerns (implements Application interfaces)
│   ├── Data/
│   │   ├── ApplicationDbContext.cs  # EF Core DbContext
│   │   ├── Configurations/          # Entity configurations
│   │   │   ├── DrugConfiguration.cs      # Central catalog config
│   │   │   ├── ShopConfiguration.cs      # Shop entities
│   │   │   └── InventoryConfiguration.cs # Per-shop inventory
│   │   ├── Repositories/    # EF Core implementations
│   │   └── Migrations/      # Database migrations
│   └── SampleData/          # Test data generators
└── API/                     # HTTP layer
    ├── Controllers/         # REST endpoints (use MediatR, NOT repositories directly)
    │   ├── DrugsController.cs      # Central drug catalog APIs
    │   ├── ShopsController.cs      # Shop registration & management
    │   └── InventoryController.cs  # Per-shop inventory management
    └── Extensions/          # DI configuration per layer
```

### Multi-Tenant Data Model

**Shared Data (All Shops)**:

- Drug catalog (DrugId, BrandName, GenericName, Barcode, Formulation, Regulatory)
- Manufacturer information
- Supplier contacts

**Shop-Specific Data**:

- Inventory (TotalStock, ReorderPoint, Batches per shop)
- Local pricing (can override suggested pricing)
- Orders and transactions
- Customer data (per shop)
- Shop profile (name, address, license)

## Critical Patterns

### 1. CQRS with MediatR (ALWAYS use this pattern)

**Queries (Read):**

```csharp
// 1. Define query as record
public record GetDrugQuery(string DrugId) : IRequest<DrugDto?>;

// 2. Create handler in same folder
public class GetDrugQueryHandler : IRequestHandler<GetDrugQuery, DrugDto?>
{
    private readonly IDrugRepository _repository;

    public async Task<DrugDto?> Handle(GetDrugQuery request, CancellationToken ct)
    {
        var entity = await _repository.GetByIdAsync(request.DrugId, ct);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [halgurdkamal/pos_system_api](https://github.com/halgurdkamal/pos_system_api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
