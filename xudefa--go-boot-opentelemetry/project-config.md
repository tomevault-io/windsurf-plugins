---
trigger: always_on
description: go-boot-opentelemetry 是一个基于 [github.com/xudefa/go-boot](https://github.com/xudefa/go-boot) 的 OpenTelemetry 分布式追踪集成模块。本模块将 OpenTelemetry SDK 无缝集成到 go-boot 的 IoC 容器和自动配置体系中，遵循 go-boot 项目的开发规范。
---

# go-boot-opentelemetry 项目开发规范文档

go-boot-opentelemetry 是一个基于 [github.com/xudefa/go-boot](https://github.com/xudefa/go-boot) 的 OpenTelemetry 分布式追踪集成模块。本模块将 OpenTelemetry SDK 无缝集成到 go-boot 的 IoC 容器和自动配置体系中，遵循 go-boot 项目的开发规范。

## 1. 项目定位

### 1.1 与 go-boot 的关系

- **基础框架**：go-boot 提供核心 IoC 容器、AOP、自动配置、生命周期管理等基础设施
- **集成模块**：go-boot-opentelemetry 是 go-boot 的追踪层集成，将 OpenTelemetry 作为分布式追踪实现
- **规范继承**：完全遵循 go-boot 的开发规范、命名约定、代码风格

### 1.2 核心职责

- 将 OpenTelemetry TracerProvider 和 Tracer 注册为 go-boot 容器中的 Bean
- 提供基于 go-boot 自动配置的 OpenTelemetry 启动器
- 实现 `tracing.Tracer` 和 `tracing.Span` 接口的 OpenTelemetry 适配器
- 支持 OTLP gRPC、OTLP HTTP、Stdout 等多种导出器
- 支持 W3C TraceContext 和 Baggage 上下文传播

## 2. 项目架构

### 2.1 整体架构

```
┌───────────────────────────────────────────────────────────────────────┐
│                    go-boot ApplicationContext                         │
│  ┌───────────┐ ┌──────────────┐ ┌───────────┐ ┌───────────┐           │
│  │ Container │ │  Environment │ │ Lifecycle │ │ EventBus  │           │
│  └───────────┘ └──────────────┘ └───────────┘ └───────────┘           │
│                       ┌─────────────────────┐                         │
│                       │ AutoConfig Registry │                         │
│                       └─────────────────────┘                         │
└───────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                    ┌───────────────────────────────┐
                    │ go-boot-opentelemetry Starter │
                    │  ┌─────────────────────────┐  │
                    │  │ TracerProvider Bean     │  │
                    │  │ Tracer Bean             │  │
                    │  │ Span Wrapper            │  │
                    │  │ Exporter (gRPC/HTTP)    │  │
                    │  │ Propagator              │  │
                    │  └─────────────────────────┘  │
                    └───────────────────────────────┘
```

- **基础依赖**：依赖 go-boot 核心框架（`github.com/xudefa/go-boot`）
- **追踪系统**：集成 OpenTelemetry（`go.opentelemetry.io/otel`）
- **职责边界**：仅负责分布式追踪集成，不包含其他业务逻辑
- **示例代码**：统一放在 `examples/` 目录，演示 OpenTelemetry 集成用法

### 2.2 go-boot-opentelemetry 核心包结构

| 包 | 说明 | 接口定义 |
|---|------|----------|
| `autoconfig.go` | OpenTelemetry 自动配置 | `AutoConfiguration` |
| `tracer_adapter.go` | Tracer/Span 适配器 | `TracerWrapper`, `SpanWrapper` |
| `propagator.go` | 上下文传播器 | `Propagator` |
| `bridge/` | 追踪桥接实现 | `TracingBridge` |

### 2.3 go-boot 核心包参考

go-boot-opentelemetry 依赖 go-boot 的以下核心包：

| 包 | 说明 | 接口定义 |
|---|------|----------|
| `core/` | IoC 容器（依赖注入核心） | `core.Container` |
| `aop/` | AOP 框架（5 种通知类型 + 多种切点匹配） | `aop.Advice`, `aop.PointCut`, `aop.Weaver` |
| `boot/` | 应用启动器、自动配置注册、横幅、失败分析 | `boot.AutoConfiguration`, `boot.Starter` |
| `context/` | 应用上下文（聚合容器、环境、生命周期、事件） | `context.ApplicationContext` |
| `environment/` | 环境配置管理（分层 PropertySource + Profile） | `environment.Environment` |
| `condition/` | 条件判断（OnProperty / OnBean / OnClass 等） | `condition.Condition` |
| `event/` | 事件驱动支持（发布/订阅） | `event.ApplicationEvent`, `event.EventPublisher` |
| `life/` | 生命周期阶段管理（7 个阶段） | `life.Lifecycle`, `life.Phase` |
| `data/` | 数据访问抽象（Repository[T] / Transactor） | `data.Repository[T]`, `data.Transactor` |
| `cache/` | 缓存抽象（Cache 接口） | `cache.Cache` |
| `config/` | 配置管理（Config 接口 + Loader 链 + Validator） | `config.Config`, `config.Loader` |
| `log/` | 日志抽象（Logger 接口 + slog 默认实现） | `log.Logger` |
| `net/` | HTTP 服务器/客户端抽象接口 | `net.Server`, `net.HttpClient` |
| `health/` | 健康指标（Indicator + Aggregator） | `health.Indicator`, `health.HealthAggregator` |
| `metrics/` | 指标收集（Counter + Gauge + Registry） | `metrics.Counter`, `metrics.Gauge`, `metrics.MeterRegistry` |
| `tracing/` | 分布式追踪抽象 + LocalTracer 实现 | `tracing.Tracer`, `tracing.Span` |
| `actuator/` | 运维端点（健康、指标、环境信息） | `actuator.Endpoint` |
| `schedule/` | 定时任务调度（Cron 解析、最小堆调度器、@Scheduled 注解） | `schedule.Task`, `schedule.Scheduler` |
| `center/` | 注册中心抽象（Registry 接口 + Selector 接口 + 内置选择器） | `center.Registry`, `center.Selector` |
| `refresh/` | 配置刷新（RefreshScope 动态刷新） | `refresh.RefreshScopeManager` |

### 2.4 接口抽象原则

go-boot-opentelemetry 遵循 go-boot 的接口抽象原则，所有集成层通过核心框架中的接口抽象定义，实现运行时互换：

- `net.Server` / `net.HttpClient` — HTTP 服务器/客户端
- `data.Repository[T]` / `data.Transactor` — 数据访问/事务
- `cache.Cache` — 缓存抽象
- `config.Config` — 配置管理
- `log.Logger` — 日志抽象
- `health.Indicator` — 健康指标
- `metrics.MeterRegistry` — 指标注册表
- `tracing.Tracer` / `tracing.Span` — 分布式追踪
- `center.Registry` / `center.Selector` — 注册发现/负载均衡
- `core.Container` — IoC 容器
- `boot.AutoConfiguration` — 自动配置
- `boot.Starter` — 启动器生命周期

## 3. 开发规范

### 3.1 命名约定

- **包名**：小写、多个单词中间用"-"连接，除开main包，其他包名和最里层目录名保持一致。例如 `user-service`
- **导出标识符**：大写驼峰（`UserID`）
- **非导出标识符**：小写驼峰（`userID`）
- **常量**：使用驼峰，而非全大写加下划线（`MaxConnections` 而不是 `MAX_CONNECTIONS`）
- **测试函数**：`TestFunctionName_Condition_ExpectedBehavior`
- **错误变量**：以 `Err` 前缀（`ErrNotFound`）
- **接口**：通常以 `er` 后缀（`Reader`, `Writer`）或功能描述（`Logger`, `Cache`）

### 3.2 导入规范

- 使用标准库分组 → 本地包，每组之间用空白行分隔
- 禁止相对导入（如 `../foo`），使用模块路径完整导入
- 核心框架仅使用 Go 标准库

```go
import (
    "context"
    "fmt"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xudefa/go-boot-opentelemetry](https://github.com/xudefa/go-boot-opentelemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
