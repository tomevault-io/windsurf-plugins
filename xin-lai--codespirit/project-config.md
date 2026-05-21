---
trigger: always_on
description: CodeSpirit 统一启动框架规范 - API项目配置标准化
---


# 统一启动框架规范

## 快速开始

### Program.cs（标准模板）
```csharp
using CodeSpirit.ExamApi.Configuration;
using CodeSpirit.Shared.Startup;
using System.Text;

Console.OutputEncoding = Encoding.UTF8;

var builder = WebApplication.CreateBuilder(args);

// 1. 注册服务
builder.AddCodeSpiritApi<ExamApiConfiguration>();

var app = builder.Build();

try
{
    // 2. 配置中间件和初始化数据库
    await app.UseCodeSpiritApiAsync<ExamApiConfiguration>();
    app.Run();
}
catch (Exception ex)
{
    var logger = app.Services.GetRequiredService<ILogger<Program>>();
    logger.LogError(ex, "服务启动过程中发生错误");
}
```

> ⚠️ **规范**：不要在 `Program.cs` 中添加额外配置，所有配置放在 API 配置类中。

---

## API 配置类

### 位置和命名
- **位置**：`{ProjectName}/Configuration/` 文件夹
- **命名**：`{ApiName}Configuration`（如 `ExamApiConfiguration`）
- **继承**：`BaseApiConfiguration`

### 最小配置类
```csharp
namespace CodeSpirit.ExamApi.Configuration;

public class ExamApiConfiguration : BaseApiConfiguration
{
    /// <summary>服务名称，用于 Aspire 服务发现</summary>
    public override string ServiceName => "exam";
    
    /// <summary>数据库连接字符串键名</summary>
    public override string ConnectionStringKey => "exam-api";
}
```

---

## 核心方法

### ConfigureServices - 服务注册

#### 简化配置方式（推荐）

使用扩展方法简化配置，减少重复代码：

```csharp
public override void ConfigureServices(IServiceCollection services, IConfiguration configuration)
{
    // ⚠️ 必须调用基类方法（初始化路径前缀配置）
    base.ConfigureServices(services, configuration);
    
    // 配置标准数据库服务（多数据库支持、仓储模式）
    this.ConfigureStandardDatabaseServices<ExamDbContext, MySqlExamDbContext, SqlServerExamDbContext>(
        services, configuration);
    
    // 配置标准基础设施服务（事件总线、HTTP客户端）+ 可选组件（多租户、设置管理）
    this.ConfigureStandardInfrastructureServices(services, configuration, (s, c) =>
    {
        s.AddCodeSpiritMultiTenant(c);
        s.AddSettingsManagerWithDatabase(c);
    });
    
    // 只配置特定业务服务
    services.AddLLMServices();
    AddExamSpecificServices(services);
}
```

#### 传统配置方式

如果需要更多控制，可以使用传统方式：

```csharp
public override void ConfigureServices(IServiceCollection services, IConfiguration configuration)
{
    // ⚠️ 必须调用基类方法（初始化路径前缀配置）
    base.ConfigureServices(services, configuration);
    
    // 配置多数据库支持（推荐方式）
    DatabaseMigrationHelper.ConfigureMultiDatabaseDbContext<
        ExamDbContext, MySqlExamDbContext, SqlServerExamDbContext>(
        services, configuration, ConnectionStringKey);
    
    // 注册仓储模式
    services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
    
    // 添加多租户支持
    services.AddCodeSpiritMultiTenant(configuration);
    
    // 其他服务注册...
}
```

#### 常用服务注册方法
| 方法 | 说明 |
|------|------|
| `services.AddCodeSpiritMultiTenant(configuration)` | 多租户支持 |
| `services.AddEventBus()` | 事件总线 |
| `services.AddCodeSpiritCaching(configuration)` | 统一缓存服务 |
| `services.AddLLMServices()` | LLM 服务 |
| `services.AddAiFormFillEndpoints()` | AI 表单填充端点 |
| `services.AddSettingsManagerWithDatabase(configuration)` | 设置管理 |
| `services.AddScheduledTasks()` | 定时任务 |
| `services.AddChartServices()` | 图表服务 |

#### 线程池配置（可选）
```csharp
// 高并发服务
ThreadPoolConfiguration.ConfigureThreadPool(
    ThreadPoolConfiguration.ServiceTier.High, 
    expectedInstances: 3, 
    logger);

// 服务等级：Low, Medium, High
```

---

### 中间件配置

#### 1. ConfigurePreAuthenticationMiddlewareAsync - 认证前
在认证之前执行，用于租户解析等：
```csharp
public override Task ConfigurePreAuthenticationMiddlewareAsync(WebApplication app)
{
    app.UseCodeSpiritMultiTenant();
    return Task.CompletedTask;
}
```

#### 2. ConfigurePreControllerMiddlewareAsync - 控制器前
在控制器映射之前执行：
```csharp
public override Task ConfigurePreControllerMiddlewareAsync(WebApplication app)
{
    // 通常审计由网关处理，API 服务不需要
    return Task.CompletedTask;
}
```

#### 3. ConfigureMiddlewareAsync - 自定义中间件
在通用中间件之后执行：

**简化配置方式（推荐）：**

```csharp
public override async Task ConfigureMiddlewareAsync(WebApplication app)
{
    // 配置标准中间件（聚合器）+ 可选组件（多租户、AI表单填充）
    await this.ConfigureStandardMiddlewareAsync(app, a =>
    {
        a.UseCodeSpiritMultiTenant();
        a.UseAiFormFillEndpoints();
    });
    
    // 只配置特定中间件
    app.MapHub<ExamHub>("/exam-hub");
}
```

**传统配置方式：**

```csharp
public override async Task ConfigureMiddlewareAsync(WebApplication app)
{
    // 多租户中间件
    app.UseCodeSpiritMultiTenant();
    
    // 聚合器中间件
    app.UseCodeSpiritAggregator();
    
    // AI 表单填充端点
    app.UseAiFormFillEndpoints();
    
    // SignalR Hub 映射
    app.MapHub<ExamHub>("/exam-hub");
    
    await Task.CompletedTask;
}
```

---

### InitializeDatabaseAsync - 数据库初始化

**简化配置方式（推荐）：**

```csharp
public override async Task InitializeDatabaseAsync(WebApplication app)
{
    // 使用标准数据库初始化方法
    // 自动应用迁移和初始化种子数据（如果 DbContext 实现了 IInitializableDbContext）
    await this.InitializeStandardDatabaseAsync<ExamDbContext, MySqlExamDbContext, SqlServerExamDbContext>(
        app, "ExamApi");
}
```

**传统配置方式：**

```csharp
public override async Task InitializeDatabaseAsync(WebApplication app)
{
    using var scope = app.Services.CreateScope();
    var services = scope.ServiceProvider;
    var logger = services.GetRequiredService<ILogger<ExamApiConfiguration>>();
    var configuration = services.GetRequiredService<IConfiguration>();
    
    try
    {
        // 1. 自动应用数据库迁移
        await DatabaseMigrationHelper.ApplyDatabaseMigrationsAsync<
            MySqlExamDbContext, 
            SqlServerExamDbContext>(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
