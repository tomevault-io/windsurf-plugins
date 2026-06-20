---
trigger: always_on
description: Application context (ASP.NET Core, Console App, Test, etc.)
---


# Myth Ecosystem - Integration Guide

**Version:** 1.0
**Target Framework:** .NET 8.0 / .NET 10.0
**License:** Apache 2.0

> 📚 **For detailed API documentation**, refer to individual library SKILL.md files in each project folder.

---

## Table of Contents

1. [Overview](#overview)
2. [The 12 Libraries](#the-12-libraries)
3. [Quick Start](#quick-start)
4. [Integration Patterns](#integration-patterns)
5. [Complete Workflows](#complete-workflows)
6. [Best Practices](#best-practices)

---

## Overview

**Myth** is a comprehensive ecosystem of 12 .NET libraries designed to work together seamlessly, providing enterprise-grade capabilities for building scalable, maintainable applications following SOLID principles, Clean Architecture, and Domain-Driven Design.

### Core Philosophy

- **Modular**: Use only what you need
- **Composable**: Libraries integrate naturally
- **Type-Safe**: Full compile-time checking
- **Async-First**: Built for modern .NET async/await
- **DI-Native**: Deep integration with Microsoft.Extensions.DependencyInjection
- **Production-Ready**: Battle-tested patterns and resilience

### Key Architectural Concept: Global Service Provider

All Myth libraries share a centralized `MythServiceProvider` for seamless dependency resolution across libraries:

```csharp
// ASP.NET Core - use BuildApp() instead of Build()
var app = builder.BuildApp(); // ✅ Initializes global provider

// Console Apps - use BuildWithGlobalProvider()
var serviceProvider = services.BuildWithGlobalProvider(); // ✅

// Now all libraries can resolve dependencies from each other
```

---

## The 12 Libraries

### Core Foundation

| Library | Purpose | Documentation |
|---------|---------|---------------|
| **Myth.Commons** | Base types, ValueObjects, JSON extensions, global ServiceProvider | [SKILL.md](Myth.Commons/SKILL.md) |
| **Myth.DependencyInjection** | Auto-discovery, convention-based service registration | [SKILL.md](Myth.DependencyInjection/SKILL.md) |

### Data & Persistence

| Library | Purpose | Documentation |
|---------|---------|---------------|
| **Myth.Repository** | Generic repository interfaces with read/write separation | [SKILL.md](Myth.Repository/SKILL.md) |
| **Myth.Repository.EntityFramework** | EF Core implementation with Unit of Work, auto-configuration | [SKILL.md](Myth.Repository.EntityFramework/SKILL.md) |
| **Myth.Specification** | Query specification pattern for encapsulating business rules | [SKILL.md](Myth.Specification/SKILL.md) |

### Validation & Transformation

| Library | Purpose | Documentation |
|---------|---------|---------------|
| **Myth.Guard** | Fluent validation with 100+ rules, context-aware, RFC 9457 errors | [SKILL.md](Myth.Guard/SKILL.md) |
| **Myth.Morph** | Object transformation and mapping with schema-based bindings | [SKILL.md](Myth.Morph/SKILL.md) |

### Workflows & Architecture Patterns

| Library | Purpose | Documentation |
|---------|---------|---------------|
| **Myth.Flow** | Pipeline pattern with Result, retry policies, telemetry | [SKILL.md](Myth.Flow/SKILL.md) |
| **Myth.Flow.Actions** | CQRS, event-driven architecture, message brokers (extends Flow) | [SKILL.md](Myth.Flow.Actions/SKILL.md) |

### HTTP & External Services

| Library | Purpose | Documentation |
|---------|---------|---------------|
| **Myth.Rest** | Fluent REST client with circuit breaker and retry policies | [SKILL.md](Myth.Rest/SKILL.md) |

### Testing & Code Generation

| Library | Purpose | Documentation |
|---------|---------|---------------|
| **Myth.Testing** | Testing utilities, mocks, base test classes (xUnit, Bogus, Moq) | [SKILL.md](Myth.Testing/SKILL.md) |
| **Myth.Tool** | CLI tool for scaffolding CQRS, DDD, and Clean Architecture patterns | [SKILL.md](Myth.Tool/SKILL.md) |

---

## Quick Start

### ASP.NET Core Application (Full Stack)

```csharp
using Myth.Extensions;

var builder = WebApplication.CreateBuilder(args);

// 1. Configure Myth libraries
builder.Services.AddFlow(config => config
    .UseTelemetry()
    .UseRetry(maxAttempts: 3, backoffMs: 1000)
    .UseActions(actions => actions
        .UseRabbitMQ(opts => { /* configure */ })
        .UseCaching(cache => cache.UseRedis("localhost:6379"))
        .ScanAssemblies(typeof(Program).Assembly)
        .AutoSubscribeEventHandlers()));

builder.Services.AddGuard();
builder.Services.AddMorph();

// 2. Configure database
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

// 3. Register repositories
builder.Services.AddRepositories(); // Auto-registers all repositories
builder.Services.AddUnitOfWorkForContext<AppDbContext>();

// 4. CRITICAL: Build with global provider
var app = builder.BuildApp(); // ✅ NOT builder.Build()

// 5. Add middleware
app.UseGuard(); // Validation exception handling with RFC 9457 format

app.MapControllers();
app.Run();
```

### Console Application (Minimal)

```csharp
using Myth.Extensions;

var services = new ServiceCollection();

services.AddFlow();
services.AddGuard();
services.AddLogging();

// Register your services
services.AddScoped<IDataProcessor, DataProcessor>();

// CRITICAL: Build with global provider
var serviceProvider = services.BuildWithGlobalProvider();

// Use services

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulaolileal/myth](https://github.com/paulaolileal/myth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
