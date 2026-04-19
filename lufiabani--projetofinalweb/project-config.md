---
trigger: always_on
description: - Siga APENAS os padrões ensinados pelo professor Matheus Cataneo
---

# .cursorrules — Desenvolvimento de Sistemas Web
# Professor: Matheus Cataneo
# IMPORTANTE: Siga EXATAMENTE os padrões ensinados pelo professor.
# NÃO use padrões alternativos, mesmo que "melhores". Simplicidade e fidelidade ao modelo.

## REGRA GERAL
- Siga APENAS os padrões ensinados pelo professor Matheus Cataneo
- NÃO adicione técnicas, bibliotecas ou padrões não ensinados
- NÃO use Repository Pattern, DTOs, AutoMapper, MediatR, FluentValidation
- NÃO use TypeScript no frontend — apenas JavaScript puro (.jsx, .js)
- Mantenha o código simples, comentado e fiel ao modelo do professor
- Comentários devem ser em português, explicando o "por quê" como o professor faz

---

## BACKEND — .NET 8 Web API + PostgreSQL

### Stack obrigatória
- .NET 8 Web API
- Entity Framework Core 8.0.0
- Npgsql.EntityFrameworkCore.PostgreSQL 8.0.0
- Microsoft.EntityFrameworkCore.Design 8.0.0
- Swashbuckle.AspNetCore.SwaggerGen 8.0.0
- Swashbuckle.AspNetCore.SwaggerUI 8.0.0

### Criação do projeto
```
dotnet new webapi -n NomeProjeto.Api --no-openapi
dotnet add package Microsoft.EntityFrameworkCore --version 8.0.0
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 8.0.0
dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.0.0
```
Instalar Swagger manualmente via csproj (SwaggerGen + SwaggerUI versão 8.0.0).
Deletar WeatherForecast.cs e WeatherForecastController.cs ao criar o projeto.

### Estrutura de pastas obrigatória
```
NomeProjeto.Api/
├── Controllers/
├── Data/
│   └── AppDbContext.cs
├── Models/
├── Migrations/       ← gerada automaticamente
├── appsettings.json
├── Program.cs
└── NomeProjeto.Api.csproj
```

### Padrão de Models
- Chave primária: public int Id { get; set; }
- Campo obrigatório: public required string Nome { get; set; }
- Campo opcional: public string? Descricao { get; set; }
- Valor monetário: public decimal Preco { get; set; }
- Data de criação: public DateTime DataCriacao { get; set; } = DateTime.UtcNow;
- SEMPRE usar DateTime.UtcNow, NUNCA DateTime.Now
- Chave estrangeira obrigatória: public int CategoriaId { get; set; }
- Chave estrangeira opcional: public int? CategoriaId { get; set; }
- Propriedade de navegação (lado "muitos"): public Categoria? Categoria { get; set; }
- Propriedade de navegação (lado "um", 1-para-N): public ICollection<Produto> Produtos { get; set; } = new List<Produto>();
- Propriedade de navegação (1-para-1): public DetalheProduto? DetalheProduto { get; set; }
- NÃO usar Data Annotations ([Required], [MaxLength], etc.) — usar Fluent API no DbContext

### Padrão do AppDbContext
```csharp
// Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using NomeProjeto.Api.Models;

namespace NomeProjeto.Api.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    public DbSet<Produto> Produtos { get; set; }
    public DbSet<Categoria> Categorias { get; set; }
    public DbSet<DetalheProduto> DetalhesProduto { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Relacionamento 1-para-N: Produto → Categoria
        modelBuilder.Entity<Produto>()
            .HasOne(p => p.Categoria)
            .WithMany(c => c.Produtos)
            .HasForeignKey(p => p.CategoriaId)
            .OnDelete(DeleteBehavior.Restrict);

        // Relacionamento 1-para-1: DetalheProduto → Produto
        modelBuilder.Entity<DetalheProduto>()
            .HasOne(d => d.Produto)
            .WithOne(p => p.DetalheProduto)
            .HasForeignKey<DetalheProduto>(d => d.ProdutoId)
            .OnDelete(DeleteBehavior.Cascade);

        // OBRIGATÓRIO para 1-para-1: índice UNIQUE na FK
        modelBuilder.Entity<DetalheProduto>()
            .HasIndex(d => d.ProdutoId)
            .IsUnique();
    }
}
```

### Regras do DbContext
- Relacionamento 1-para-N: usar HasOne/WithMany + OnDelete(DeleteBehavior.Restrict)
- Relacionamento 1-para-1: usar HasOne/WithOne + HasForeignKey<T> + OnDelete(DeleteBehavior.Cascade) + .HasIndex().IsUnique()
- Restrict no 1-para-N: protege contra deleção acidental de registros pai com filhos
- Cascade no 1-para-1: detalhe sem entidade principal não faz sentido, deleta junto
- NUNCA usar EntityState.Modified no PUT

### Padrão do Program.cs
```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;
using NomeProjeto.Api.Data;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        // Evita erro de ciclo de serialização em relacionamentos
        options.JsonSerializerOptions.ReferenceHandler =
            System.Text.Json.Serialization.ReferenceHandler.IgnoreCycles;
    });

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddCors(options =>
{
    options.AddPolicy("PermitirTudo", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lufiabani) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
