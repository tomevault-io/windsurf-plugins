---
trigger: always_on
description: CodeSpirit 数据库与 EF Core 迁移规范 - 多数据库支持、DbContext 设计、迁移命令
---


# 数据库与 EF Core 迁移规范

## 📋 目录

1. [多数据库架构](#多数据库架构)
2. [DbContext 设计模式](#dbcontext-设计模式)
3. [迁移命令规范](#迁移命令规范)
4. [实体 ID 配置](#实体-id-配置)
5. [实体配置规范](#实体配置规范)

---

## 多数据库架构

CodeSpirit 支持多数据库（SQL Server / MySQL），每个 API 服务需要定义：

```
Data/
├── {Service}DbContext.cs              # 基础 DbContext（运行时使用）
├── SqlServer{Service}DbContext.cs     # SQL Server 专用 DbContext
├── SqlServer{Service}DbContextFactory.cs  # SQL Server 设计时工厂
├── MySql{Service}DbContext.cs         # MySQL 专用 DbContext
├── MySql{Service}DbContextFactory.cs  # MySQL 设计时工厂
├── DatabaseSpecificConfigurations.cs  # 数据库特定配置
├── Configurations/                    # 实体配置
│   └── {Entity}Configuration.cs
└── Migrations/
    ├── SqlServer/                     # SQL Server 迁移
    │   └── {timestamp}_{MigrationName}.cs
    └── MySql/                         # MySQL 迁移
        └── {timestamp}_{MigrationName}.cs
```

---

## DbContext 设计模式

### 基础 DbContext

运行时使用的 DbContext，继承自 `MultiDatabaseDbContextBase`：

```csharp
public class MallDbContext : MultiDatabaseDbContextBase
{
    public MallDbContext(
        DbContextOptions options,
        IServiceProvider serviceProvider,
        ICurrentUser currentUser,
        IHttpContextAccessor httpContextAccessor) 
        : base(options, serviceProvider, currentUser, httpContextAccessor)
    {
    }

    // DbSet 属性
    public DbSet<Product> Products => Set<Product>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(MallDbContext).Assembly);
    }
}
```

### 数据库特定 DbContext

**SQL Server 版本**：

```csharp
/// <summary>
/// SQL Server 特定的数据库上下文（用于迁移）
/// </summary>
public class SqlServerMallDbContext : MallDbContext
{
    public SqlServerMallDbContext(
        DbContextOptions<SqlServerMallDbContext> options,
        IServiceProvider serviceProvider,
        ICurrentUser currentUser,
        IHttpContextAccessor httpContextAccessor) 
        : base((DbContextOptions)options, serviceProvider, currentUser, httpContextAccessor)
    {
    }

    protected override void ApplyDatabaseSpecificConfigurations(ModelBuilder modelBuilder)
    {
        DatabaseSpecificConfigurations.ApplySqlServerConfigurations(modelBuilder);
    }
}
```

**MySQL 版本**：

```csharp
/// <summary>
/// MySQL 特定的数据库上下文（用于迁移）
/// </summary>
public class MySqlMallDbContext : MallDbContext
{
    public MySqlMallDbContext(
        DbContextOptions<MySqlMallDbContext> options,
        IServiceProvider serviceProvider,
        ICurrentUser currentUser,
        IHttpContextAccessor httpContextAccessor) 
        : base((DbContextOptions)options, serviceProvider, currentUser, httpContextAccessor)
    {
    }

    protected override void ApplyDatabaseSpecificConfigurations(ModelBuilder modelBuilder)
    {
        DatabaseSpecificConfigurations.ApplyMySqlConfigurations(modelBuilder);
    }
}
```

### 设计时工厂

用于 `dotnet ef` 命令的设计时 DbContext 创建：

**SQL Server**:

```csharp
public class SqlServerMallDbContextFactory : IDesignTimeDbContextFactory<SqlServerMallDbContext>
{
    public SqlServerMallDbContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<SqlServerMallDbContext>();
        
        optionsBuilder.UseSqlServer(
            "Server=localhost;Database=Mall;User Id=sa;Password=Password123!;TrustServerCertificate=True;",
            options => options.MigrationsHistoryTable("__EFMigrationsHistory", "mall")
        );

        var services = new ServiceCollection();
        var serviceProvider = services.BuildServiceProvider();
        var currentUser = new DesignTimeCurrentUser();
        var httpContextAccessor = new HttpContextAccessor();

        return new SqlServerMallDbContext(optionsBuilder.Options, serviceProvider, currentUser, httpContextAccessor);
    }
}
```

**MySQL**:

```csharp
public class MySqlMallDbContextFactory : IDesignTimeDbContextFactory<MySqlMallDbContext>
{
    public MySqlMallDbContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<MySqlMallDbContext>();
        
        optionsBuilder.UseMySql(
            "Server=localhost;Database=Mall;User=root;Password=password;",
            new MySqlServerVersion(new Version(8, 0, 21)),
            options => options.MigrationsHistoryTable("__EFMigrationsHistory")
        );

        var services = new ServiceCollection();
        var serviceProvider = services.BuildServiceProvider();
        var currentUser = new DesignTimeCurrentUser();
        var httpContextAccessor = new HttpContextAccessor();

        return new MySqlMallDbContext(optionsBuilder.Options, serviceProvider, currentUser, httpContextAccessor);
    }
}
```

---

## 迁移命令规范

### ⚠️ 重要：必须使用数据库特定的 DbContext

**❌ 错误**：使用基础 DbContext
```bash
dotnet ef migrations add InitialCreate --context MallDbContext
```

**✅ 正确**：使用数据库特定的 DbContext

#### SQL Server 迁移

```bash
# 创建迁移
dotnet ef migrations add InitialCreate --context SqlServerMallDbContext --output-dir Data/Migrations/SqlServer

# 更新数据库
dotnet ef database update --context SqlServerMallDbContext

# 删除最后一次迁移
dotnet ef migrations remove --context SqlServerMallDbContext
```

#### MySQL 迁移

```bash
# 创建迁移

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
