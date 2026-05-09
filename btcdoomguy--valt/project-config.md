---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Valt is a personal budget management desktop application for bitcoiners, built with .NET 9 and Avalonia UI. It tracks fiat and bitcoin accounts, transactions, and displays values in bitcoin terms.

## Module Documentation

For detailed documentation on specific modules, see:
- **[Budget Module](.claude/docs/budget.md)** - Accounts, transactions, categories
- **[Reports Module](.claude/docs/reports.md)** - Financial analysis, charts, dashboards
- **[AvgPrice Module](.claude/docs/avgprice.md)** - Cost basis tracking (BrazilianRule, FIFO)
- **[Goals Module](.claude/docs/goals.md)** - Financial goal tracking with auto-progress
- **[Fixed Expenses Module](.claude/docs/fixedexpenses.md)** - Recurring expense management
- **[Assets Module](.claude/docs/assets.md)** - External investments (stocks, ETFs, crypto, real estate, leveraged positions)

## Build and Run Commands

```bash
dotnet build Valt.sln                    # Build solution
dotnet run --project src/Valt.UI/Valt.UI.csproj  # Run application
dotnet test                              # Run all tests
dotnet test --filter "FullyQualifiedName~Valt.Tests.Domain.Budget.Transactions.TransactionTests"  # Specific test file
```

## Architecture

### Layered Structure

- **Valt.Core** - Domain layer: entities, value objects, domain events. No external dependencies.
  - `Kernel/` - Base classes (`Entity<T>`, `AggregateRoot<T>`), events, ID generation
  - `Modules/Budget/` - Accounts, Categories, Transactions, FixedExpenses
  - `Modules/AvgPrice/` - Cost basis tracking
  - `Modules/Goals/` - Financial goals
  - `Modules/Assets/` - External investments
  - `Common/` - Value objects (`BtcValue`, `FiatValue`, `FiatCurrency`, `Icon`)

- **Valt.App** - Application layer: CQRS commands, queries, and business orchestration. See [CQRS Architecture](#cqrs-architecture).
  - `Kernel/` - Command/Query dispatchers, validation, Result type
  - `Modules/Budget/` - Account, Category, Transaction, FixedExpense commands and queries
  - `Modules/AvgPrice/` - Cost basis profile commands and queries
  - `Modules/Goals/` - Goal commands and queries
  - `Modules/Assets/` - Asset commands and queries

- **Valt.Infra** - Infrastructure layer: persistence, external services
  - `DataAccess/` - LiteDB database access
  - `Modules/` - Repositories, queries, reports
  - `Crawlers/` - Price providers (Kraken, Coinbase, Frankfurter)
  - `Kernel/BackgroundJobs/` - Periodic tasks
  - `Services/` - Updates, CSV import
  - `Mcp/` - MCP server and tools for AI assistant integration

- **Valt.UI** - Avalonia desktop application
  - `Views/Main/Tabs/` - Transactions, Reports, AvgPrice tabs
  - `Views/Main/Modals/` - Modal dialogs
  - `Base/` - ViewModel base classes
  - `State/` - Application state (RatesState, AccountsTotalState, FilterState)

### Key Patterns

- **MVVM**: ViewModels inherit from `ValtViewModel` (CommunityToolkit.Mvvm)
- **CQRS**: Commands and Queries through `ICommandDispatcher` and `IQueryDispatcher`
- **Domain Events**: Aggregate roots emit events via `AddEvent()`, published through `IDomainEventPublisher`
- **Factory Pattern**: `IModalFactory`, `IPageFactory` for view creation
- **Strategy Pattern**: `IAvgPriceCalculationStrategy`, `IGoalProgressCalculator`
- **Weak Messaging**: `WeakReferenceMessenger` for loosely coupled updates

### CQRS Architecture

**IMPORTANT: Always use the Valt.App layer for new modules and features.** The App layer implements CQRS (Command Query Responsibility Segregation) pattern to separate read and write operations.

#### Commands (Write Operations)

Commands modify data and are located in `Valt.App/Modules/{Module}/Commands/`:

```csharp
// Command definition
public sealed class CreateAssetCommand : ICommand<string>  // Returns asset ID
{
    public required string Name { get; init; }
    public required string CurrencyCode { get; init; }
    // ... other properties
}

// Validator
internal sealed class CreateAssetValidator : IValidator<CreateAssetCommand>
{
    public ValidationResult Validate(CreateAssetCommand command)
    {
        var errors = new List<ValidationError>();
        if (string.IsNullOrWhiteSpace(command.Name))
            errors.Add(new("Name", "Name is required"));
        return new ValidationResult(errors.Count == 0, errors);
    }
}

// Handler
internal sealed class CreateAssetHandler : ICommandHandler<CreateAssetCommand, string>
{
    public async Task<Result<string>> HandleAsync(CreateAssetCommand command, CancellationToken ct)
    {
        var validation = _validator.Validate(command);
        if (!validation.IsValid)
            return Result<string>.Failure(new Error("VALIDATION_FAILED", "...", validation.Errors));

        // Domain logic here
        return Result<string>.Success(asset.Id.Value);
    }
}
```

For commands that don't return a value, use `ICommand<Unit>` and `ICommandHandler<TCommand, Unit>`.

#### Queries (Read Operations)

Queries read data and are located in `Valt.App/Modules/{Module}/Queries/`:

```csharp
// Query definition
public sealed class GetAssetsQuery : IQuery<IReadOnlyList<AssetDTO>> { }

// Handler

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btcdoomguy/valt](https://github.com/btcdoomguy/valt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
