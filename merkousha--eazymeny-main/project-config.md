---
trigger: always_on
description: EazyMenu is a SaaS platform for restaurant management built with **Clean Architecture** principles using **.NET 9.0**.
---

# GitHub Copilot Instructions - EazyMenu Project

## 🎯 Project Overview
EazyMenu is a SaaS platform for restaurant management built with **Clean Architecture** principles using **.NET 9.0**.

**Tech Stack:**
- Framework: ASP.NET Core 9.0 MVC (No Razor Pages)
- Database: SQL Server 2022 + Entity Framework Core 9.0
- Authentication: ASP.NET Core Identity
- Patterns: CQRS (MediatR), Repository Pattern, Unit of Work
- External Services: Zarinpal (Payment), Kavenegar (SMS), QRCoder

## 🏗️ Architecture Layers

### 1. Domain Layer (`EazyMenu.Domain`)
**Purpose:** Core business logic and entities - NO external dependencies

**Rules:**
- ✅ Contains only Entities, Enums, and Domain logic
- ✅ All entities inherit from `BaseEntity` (Id, CreatedAt, UpdatedAt, IsDeleted)
- ✅ Use `IAggregateRoot` marker for aggregate roots
- ✅ NO references to Infrastructure, Application, or any external packages
- ✅ Soft delete by default (IsDeleted flag)

**Entities:**
- `ApplicationUser` - Base user (not IdentityUser - that's in Infrastructure)
- `Restaurant`, `Subscription`, `Category`, `Product`
- `Order`, `OrderItem`, `Payment`, `Reservation`, `Notification`

**Example Entity:**
```csharp
public class Restaurant : BaseEntity, IAggregateRoot
{
    public string Name { get; set; } = string.Empty;
    public Guid OwnerId { get; set; }
    public virtual ApplicationUser Owner { get; set; } = null!;
    // ...
}
```

### 2. Application Layer (`EazyMenu.Application`)
**Purpose:** Use cases, CQRS commands/queries, interfaces

**Rules:**
- ✅ Define interfaces for repositories and services
- ✅ Use CQRS pattern with MediatR (Commands for writes, Queries for reads)
- ✅ DTOs and ViewModels go here
- ✅ AutoMapper profiles for mappings
- ✅ FluentValidation for input validation
- ✅ NO direct database access - only interfaces

**Structure:**
```
Application/
├── Common/
│   ├── Interfaces/ (IRepository, IUnitOfWork, ISmsService, etc.)
│   ├── Models/ (DTOs, ViewModels)
│   └── Mappings/ (AutoMapper profiles)
├── Features/
│   ├── Restaurants/
│   │   ├── Commands/ (CreateRestaurant, UpdateRestaurant)
│   │   └── Queries/ (GetRestaurantById, GetAllRestaurants)
│   ├── Products/
│   ├── Orders/
│   └── ...
```

**Example Command:**
```csharp
public class CreateRestaurantCommand : IRequest<Guid>
{
    public string Name { get; set; } = string.Empty;
    public string PhoneNumber { get; set; } = string.Empty;
    // ...
}

public class CreateRestaurantCommandHandler : IRequestHandler<CreateRestaurantCommand, Guid>
{
    private readonly IRepository<Restaurant> _repository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task<Guid> Handle(CreateRestaurantCommand request, CancellationToken cancellationToken)
    {
        var restaurant = new Restaurant { /* map from request */ };
        await _repository.AddAsync(restaurant, cancellationToken);
        await _unitOfWork.SaveChangesAsync(cancellationToken);
        return restaurant.Id;
    }
}
```

### 3. Infrastructure Layer (`EazyMenu.Infrastructure`)
**Purpose:** Data access, external services implementation

**Rules:**
- ✅ `ApplicationDbContext` extends `IdentityDbContext<ApplicationIdentityUser, IdentityRole<Guid>, Guid>`
- ✅ Use Fluent API for entity configurations (IEntityTypeConfiguration)
- ✅ Implement repositories and UnitOfWork
- ✅ External service implementations (SMS, Payment, QRCode)
- ✅ Global query filters for soft delete
- ✅ Auto-set CreatedAt/UpdatedAt in SaveChangesAsync

**Key Files:**
- `Data/ApplicationDbContext.cs` - Main DbContext
- `Data/Configurations/` - Entity configurations
- `Repositories/Repository.cs` - Generic repository
- `Repositories/UnitOfWork.cs` - Transaction management
- `Services/` - External service implementations
- `Identity/ApplicationIdentityUser.cs` - Identity user (extends IdentityUser)

**Example Configuration:**
```csharp
public class RestaurantConfiguration : IEntityTypeConfiguration<Restaurant>
{
    public void Configure(EntityTypeBuilder<Restaurant> builder)
    {
        builder.HasKey(r => r.Id);
        builder.Property(r => r.Name).IsRequired().HasMaxLength(200);
        builder.HasIndex(r => r.Slug).IsUnique();
        
        builder.HasOne(r => r.Owner)
            .WithMany(u => u.Restaurants)
            .HasForeignKey(r => r.OwnerId)
            .OnDelete(DeleteBehavior.Restrict);
    }
}
```

### 4. Web Layer (`EazyMenu.Web`)
**Purpose:** Presentation layer - MVC controllers and views

**Rules:**
- ✅ MVC only (NO Razor Pages as per PRD)
- ✅ Use Areas for logical separation (Admin, Restaurant)
- ✅ Controllers are thin - delegate to MediatR
- ✅ Mobile-first responsive design
- ✅ NO inline styles or scripts
- ✅ Persian (RTL) + English (LTR) support

**Structure:**
```
Web/
├── Areas/
│   ├── Admin/ (System administration)
│   └── Restaurant/ (Restaurant owner panel)
├── Controllers/ (Public controllers)
├── Views/
│   ├── Shared/ (_Layout.cshtml, _AdminLayout.cshtml)
│   └── ...
├── wwwroot/
│   ├── css/
│   ├── js/
│   └── qrcodes/
```

**Example Controller:**
```csharp
[Area("Admin")]
[Authorize(Roles = "Admin")]
public class RestaurantController : Controller
{
    private readonly IMediator _mediator;
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Merkousha/EazyMeny-Main](https://github.com/Merkousha/EazyMeny-Main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
