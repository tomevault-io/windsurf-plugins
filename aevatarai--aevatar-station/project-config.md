---
trigger: always_on
description: This document contains Standard Operating Procedures (SOPs) for common unit testing scenarios in Aevatar Station. Each SOP provides step-by-step guidance to solve specific testing challenges.
---

# Aevatar Station Unit Testing SOPs

## 📖 Document Overview
This document contains Standard Operating Procedures (SOPs) for common unit testing scenarios in Aevatar Station. Each SOP provides step-by-step guidance to solve specific testing challenges.

## 🎯 How to Use This Document
1. **Find your scenario** - Use the index below to locate the relevant SOP
2. **Follow the pattern** - Each SOP follows a consistent structure for easy understanding
3. **Copy & adapt** - Code examples are ready to use in your tests
4. **Contribute** - Add new SOPs using the contribution guidelines at the end

## 📚 SOP Index

### Orleans & Distributed Systems
- [1.1 GrainTypeResolver Injection](#11-graintyperesolver-injection)
- [1.2 ClusterClient Configuration](#12-clusterclient-configuration-coming-soon) *(Coming Soon)*
- [1.3 Grain Testing Patterns](#13-grain-testing-patterns-coming-soon) *(Coming Soon)*

### Database & Persistence  
- [2.1 MongoDB Test Configuration](#21-mongodb-test-configuration-coming-soon) *(Coming Soon)*
- [2.2 Entity Framework Test Setup](#22-entity-framework-test-setup-coming-soon) *(Coming Soon)*

### External Dependencies
- [3.1 HTTP Client Mocking](#31-http-client-mocking-coming-soon) *(Coming Soon)*
- [3.2 SignalR Testing](#32-signalr-testing-coming-soon) *(Coming Soon)*

---

## 1.1 GrainTypeResolver Injection

### 🎯 Problem Description
When writing unit tests for services that depend on Orleans GrainTypeResolver, developers encounter `DependencyInjectionException` because GrainTypeResolver cannot be resolved from the DI container.

### 🔧 Core Principle
GrainTypeResolver is an Orleans runtime component that must be obtained from `IClusterClient.ServiceProvider`, not directly registered in DI container.

### ⚙️ Key Configuration

#### Test Base Module Pattern
```csharp
[DependsOn(
    typeof(AbpAutofacModule),
    typeof(AbpTestBaseModule),
    typeof(AbpAuthorizationModule),
    typeof(AevatarCQRSModule)
)]
public class AevatarOrleansTestBaseModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        // Orleans cluster setup
        context.Services.AddSingleton<ClusterFixture>();
        context.Services.AddSingleton<IClusterClient>(sp => 
            sp.GetRequiredService<ClusterFixture>().Cluster.Client);
        context.Services.AddSingleton<IGrainFactory>(sp => 
            sp.GetRequiredService<ClusterFixture>().Cluster.GrainFactory);
        
        // CRITICAL: Bridge GrainTypeResolver from Orleans to ABP DI
        context.Services.AddSingleton<GrainTypeResolver>(sp => 
            sp.GetRequiredService<IClusterClient>().ServiceProvider.GetRequiredService<GrainTypeResolver>());
        
        // Optional: Register IGrainTypeProvider if needed
        context.Services.AddSingleton<IGrainTypeProvider, AttributeGrainTypeProvider>();
    }
}
```

#### Test Module Dependencies
```csharp
[DependsOn(
    typeof(AevatarApplicationModule),
    typeof(AbpEventBusModule),
    typeof(AevatarOrleansTestBaseModule), // REQUIRED for GrainTypeResolver
    typeof(AevatarDomainTestModule)
)]
public class AevatarApplicationTestModule : AbpModule
{
    // ... module configuration
}
```

#### Test Class Injection Pattern
```csharp
public abstract class AgentServiceTests<TStartupModule> : AevatarApplicationTestBase<TStartupModule>
    where TStartupModule : IAbpModule
{
    private readonly IClusterClient _clusterClient;
    private readonly GrainTypeResolver _grainTypeResolver;

    protected AgentServiceTests()
    {
        _clusterClient = GetRequiredService<IClusterClient>();
        
        // Method 1: From DI container (recommended after proper registration)
        _grainTypeResolver = GetRequiredService<GrainTypeResolver>();
        
        // Method 2: Direct from ClusterClient (fallback)
        // _grainTypeResolver = _clusterClient.ServiceProvider.GetRequiredService<GrainTypeResolver>();
    }
}
```

### 🔗 Module Hierarchy
```
AevatarApplicationTestBase<T>
  ↓
AevatarTestBase<T> 
  ↓
AbpIntegratedTest<T>
```

### ❗ Common Issues & Solutions

#### "Unable to resolve GrainTypeResolver"
- Ensure test module depends on `AevatarOrleansTestBaseModule`
- Verify ClusterFixture is properly configured
- Check inheritance chain is correct

#### ClusterFixture Issues
- Check Orleans cluster configuration
- Ensure no port conflicts in test environment
- Verify ClusterFixture implementation

### 💡 Best Practices
1. **Module Organization**: Keep Orleans test configuration in separate module
2. **Dependency Order**: Orleans test base module must load before application module
3. **Resource Management**: Reuse Orleans cluster instances, clean up test resources
4. **Error Handling**: Add null checks and meaningful test skip logic

### 📦 Required Imports
```csharp
using Orleans;
using Orleans.Metadata;
using Microsoft.Extensions.DependencyInjection;
using Volo.Abp.Modularity;
```

### ✅ Quick Checklist
- [ ] Test module depends on `AevatarOrleansTestBaseModule`
- [ ] GrainTypeResolver registered from ClusterClient.ServiceProvider
- [ ] Test class inherits from correct base class
- [ ] ClusterFixture properly configured

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aevatarAI/aevatar-station](https://github.com/aevatarAI/aevatar-station) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
