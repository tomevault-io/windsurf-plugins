---
trigger: always_on
description: Estamos a construir uma **web app SaaS** para eletricistas, técnicos de AVAC e pequenas empresas de manutenção em Portugal. O produto resolve problemas reais e concretos:
---

# ⚡ TécnicoApp — Skill / Especificação de Projeto para Claude Code

## Visão Geral

Estamos a construir uma **web app SaaS** para eletricistas, técnicos de AVAC e pequenas empresas de manutenção em Portugal. O produto resolve problemas reais e concretos:

- Perda de tempo a criar orçamentos à mão ou no Excel
- Falta de histórico de clientes e equipamentos
- Esquecimento de manutenções periódicas
- Falta de profissionalismo no contacto com o cliente (sem assinatura digital, sem PDF, etc.)

O target é o técnico individual ou empresa com 1-10 funcionários que paga €20-60/mês.

---

## Stack Técnica

| Camada | Tecnologia |
|---|---|
| **Frontend** | Next.js 14+ (App Router) + Tailwind CSS + shadcn/ui |
| **Backend / API** | ASP.NET Core 8 Web API (C#) |
| **Base de dados** | PostgreSQL |
| **ORM** | Entity Framework Core 8 |
| **Autenticação** | ASP.NET Core Identity + JWT Bearer |
| **Armazenamento de ficheiros** | Azure Blob Storage ou AWS S3 |
| **Geração de PDFs** | QuestPDF (C#) |
| **Email transacional** | Resend (via HTTP) ou SendGrid |
| **Jobs / Agendamento** | Hangfire (alertas de manutenção, emails) |
| **Pagamentos** | Stripe .NET SDK |
| **Deploy Backend** | Azure App Service ou Railway |
| **Deploy Frontend** | Vercel |
| **Containerização** | Docker + docker-compose para dev local |

> Para app mobile futura: React Native com Expo consumindo a mesma API .NET.

---

## Arquitetura do Backend (.NET)

### Estrutura de Pastas

```
TecnicoApp.sln
├── src/
│   ├── TecnicoApp.API/              # Projeto Web API (entry point)
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Program.cs
│   │   └── appsettings.json
│   │
│   ├── TecnicoApp.Application/      # Lógica de negócio (Use Cases)
│   │   ├── Features/
│   │   │   ├── Quotes/
│   │   │   │   ├── Commands/        # CreateQuoteCommand, UpdateQuoteCommand...
│   │   │   │   ├── Queries/         # GetQuoteByIdQuery, GetQuoteListQuery...
│   │   │   │   └── Handlers/
│   │   │   ├── Clients/
│   │   │   ├── Equipment/
│   │   │   └── Interventions/
│   │   ├── Common/
│   │   │   ├── Interfaces/          # IRepository, ICurrentUser, IEmailService...
│   │   │   ├── Behaviors/           # ValidationBehavior, LoggingBehavior (MediatR)
│   │   │   └── Exceptions/          # NotFoundException, ValidationException...
│   │   └── DTOs/
│   │
│   ├── TecnicoApp.Domain/           # Entidades, Value Objects, Enums (sem dependências)
│   │   ├── Entities/
│   │   ├── Enums/
│   │   ├── Events/                  # Domain Events
│   │   └── ValueObjects/
│   │
│   └── TecnicoApp.Infrastructure/   # Implementações concretas
│       ├── Persistence/
│       │   ├── AppDbContext.cs
│       │   ├── Configurations/      # IEntityTypeConfiguration<T>
│       │   └── Migrations/
│       ├── Services/                # EmailService, PdfService, StorageService...
│       └── Identity/
│
└── tests/
    ├── TecnicoApp.UnitTests/
    ├── TecnicoApp.IntegrationTests/
    └── TecnicoApp.FunctionalTests/
```

Esta estrutura segue **Clean Architecture** com **CQRS via MediatR**.

### Packages NuGet Principais

```xml
<!-- TecnicoApp.API -->
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.*" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="6.*" />
<PackageReference Include="Serilog.AspNetCore" Version="8.*" />

<!-- TecnicoApp.Application -->
<PackageReference Include="MediatR" Version="12.*" />
<PackageReference Include="FluentValidation.DependencyInjectionExtensions" Version="11.*" />
<PackageReference Include="AutoMapper" Version="13.*" />
<PackageReference Include="Ardalis.Result" Version="9.*" />

<!-- TecnicoApp.Infrastructure -->
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.*" />
<PackageReference Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="8.*" />
<PackageReference Include="Hangfire.AspNetCore" Version="1.*" />
<PackageReference Include="QuestPDF" Version="2024.*" />
<PackageReference Include="Stripe.net" Version="45.*" />
<PackageReference Include="Azure.Storage.Blobs" Version="12.*" />
```

---

## Entidades do Domínio (C#)

```csharp
// Domain/Entities/BaseEntity.cs
public abstract class BaseEntity
{
    public Guid Id { get; init; } = Guid.NewGuid();
    public DateTime CreatedAt { get; init; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
}

// Domain/Entities/Client.cs
public class Client : BaseEntity
{
    public string Name { get; set; } = string.Empty;
    public string? Nif { get; set; }
    public string? Email { get; set; }
    public string? Phone { get; set; }
    public Address? Address { get; set; }         // Value Object
    public string? Notes { get; set; }
    public Guid UserId { get; set; }

    public ICollection<Equipment> Equipment { get; set; } = [];
    public ICollection<Quote> Quotes { get; set; } = [];
    public ICollection<Intervention> Interventions { get; set; } = [];
}

// Domain/ValueObjects/Address.cs
public record Address(
    string Street,
    string City,
    string PostalCode,
    string Country = "Portugal"
);

// Domain/Entities/Quote.cs
public class Quote : BaseEntity
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [areiasdev/App-para-Eletricistas](https://github.com/areiasdev/App-para-Eletricistas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
