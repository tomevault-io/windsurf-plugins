---
trigger: always_on
description: CodeSpirit 依赖注入规范 - Scrutor自动注册、生命周期管理
---


# 依赖注入规范（Scrutor 自动注册）

## 概述

项目使用 Scrutor 库实现基于标记接口的自动依赖注入，无需手动注册服务。

## 标记接口

位于 `CodeSpirit.Core.DependencyInjection` 命名空间：

| 接口 | 生命周期 | 适用场景 |
|-----|---------|---------|
| `IScopedDependency` | Scoped | 业务服务、数据库操作、请求相关 |
| `ITransientDependency` | Transient | 无状态工具类、轻量操作 |
| `ISingletonDependency` | Singleton | 配置服务、缓存、ID生成器 |

### 生命周期说明

```csharp
// IScopedDependency - 作用域注入
// 同一个请求中是同一个实例，不同请求是不同实例
// 推荐：大多数业务服务、DbContext 相关操作

// ITransientDependency - 瞬时注入  
// 每次注入都创建新实例
// 推荐：无状态工具类、不持有资源的服务

// ISingletonDependency - 单例注入
// 整个应用生命周期只有一个实例
// 推荐：配置服务、缓存管理、ID生成器
```

## 标记方式

### 方式一：接口继承标记接口（推荐）

接口继承标记接口，实现类无需再次标记：

```csharp
// 接口定义 - 继承 IScopedDependency
public interface IAuthService : IScopedDependency
{
    Task<AuthResultDto> LoginAsync(LoginDto input);
    Task<bool> LogoutAsync(long userId);
}

// 实现类 - 无需标记接口
public class AuthService : IAuthService
{
    private readonly IRepository<User> _userRepository;
    
    public AuthService(IRepository<User> userRepository)
    {
        _userRepository = userRepository;
    }
    
    public async Task<AuthResultDto> LoginAsync(LoginDto input)
    {
        // 实现逻辑
    }
}
```

### 方式二：实现类标记接口

适用于无业务接口的服务类：

```csharp
// 无接口的服务类，直接实现标记接口
public class SeederService : IScopedDependency
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<SeederService> _logger;

    public SeederService(IServiceProvider serviceProvider, ILogger<SeederService> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }

    public async Task SeedAsync()
    {
        // 初始化种子数据
    }
}
```

### 方式三：同时实现业务接口和标记接口

适用于需要明确指定生命周期的服务：

```csharp
public interface IUserService : IBaseCRUDService<User, long, CreateUserDto, UpdateUserDto, UserQueryDto>
{
    Task<UserDto> GetByUsernameAsync(string username);
}

public class UserService : BaseCRUDService<User, long, CreateUserDto, UpdateUserDto, UserQueryDto>, 
    IUserService, IScopedDependency
{
    public async Task<UserDto> GetByUsernameAsync(string username)
    {
        // 实现逻辑
    }
}
```

## 生命周期选择指南

### IScopedDependency（作用域 - 最常用）

```csharp
// ✅ 业务服务
public interface IQuestionService : IScopedDependency
{
    Task<QuestionDto> GetByIdAsync(long id);
    Task CreateAsync(CreateQuestionDto dto);
}

// ✅ 数据访问服务
public interface IExamRepository : IScopedDependency
{
    Task<Exam> GetWithQuestionsAsync(long examId);
}

// ✅ 种子数据服务
public class TenantSeeder : IScopedDependency
{
    public async Task SeedAsync() { }
}
```

### ISingletonDependency（单例）

```csharp
// ✅ ID 生成器
public interface IIdGenerator : ISingletonDependency
{
    long NewId();
}

// ✅ 缓存服务
public interface IConfigCacheService : ISingletonDependency
{
    Task<string> GetAsync(string key);
    Task SetAsync(string key, string value, TimeSpan? expiry = null);
}

// ✅ 端点扫描器（应用启动时扫描一次）
public class AiFormFillEndpointScanner : ISingletonDependency
{
    public void ScanAssemblies(params Assembly[] assemblies) { }
}

// ✅ 本地化设置初始化器
public class LocalizationSettingsInitializer : ISingletonDependency
{
    public void Initialize() { }
}
```

### ITransientDependency（瞬时）

```csharp
// ✅ 无状态工具类
public interface IPasswordHasher : ITransientDependency
{
    string HashPassword(string password);
    bool VerifyPassword(string password, string hash);
}

// ✅ 存储提供器工厂（每次创建新实例）
public interface IStorageProviderFactory : ITransientDependency
{
    IStorageProvider CreateProvider(string providerType);
}

// ✅ 配置变更通知器
public interface IConfigChangeNotifier : ITransientDependency
{
    Task NotifyChangeAsync(string configKey);
}
```

## Scrutor 自动注册扩展方法

### 基础注册方法

```csharp
// 位于 CodeSpirit.Shared.DependencyInjection.ServiceCollectionExtensions

// 自动扫描并注册标记接口的服务
services.AddDependencyInjectionWithScrutor(Assembly.GetExecutingAssembly());

// 可同时扫描多个程序集
services.AddDependencyInjectionWithScrutor(
    Assembly.GetExecutingAssembly(),
    typeof(SharedService).Assembly);
```

### 高级注册方法

```csharp
// 按命名约定自动注册（Service、Repository 后缀）
services.AddAdvancedDependencyInjection(Assembly.GetExecutingAssembly());
```

### 装饰器模式

```csharp
// 使用装饰器包装现有服务
services.AddDecorator<IUserService, CachingUserServiceDecorator>();
services.AddDecorator<ILogger<UserService>, AuditLoggerDecorator<UserService>>();
```

## 注册行为

Scrutor 自动完成以下注册：

1. **接口注册**：服务注册为其实现的业务接口
2. **自身注册**：服务同时注册为自身类型（可直接注入具体类）

```csharp
// 给定服务类
public class UserService : IUserService, IScopedDependency { }

// Scrutor 自动注册：
// services.AddScoped<IUserService, UserService>();  // 接口注册
// services.AddScoped<UserService>();                 // 自身注册

// 两种方式都可以注入：
public class UserController
{
    public UserController(
        IUserService userService,      // ✅ 接口注入
        UserService userServiceImpl)   // ✅ 具体类注入
    { }
}
```

## API 配置类中的服务注册

### 自动注册（BaseApiConfiguration 已处理）

```csharp
public class ExamApiConfiguration : BaseApiConfiguration
{
    public override void ConfigureServices(IServiceCollection services, IConfiguration configuration)
    {
        base.ConfigureServices(services, configuration);
        
        // Scrutor 自动注册已在 BaseApiConfiguration 中完成
        // 无需再调用 AddDependencyInjectionWithScrutor
    }
}
```

### 手动注册特殊服务

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
