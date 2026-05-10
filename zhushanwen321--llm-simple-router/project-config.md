---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

LLM API 代理路由器。接收 OpenAI / Anthropic 格式的客户端请求，通过模型映射和路由策略转发到配置的后端 Provider，支持流式（SSE）和非流式代理。管理后台（Vue 3 + shadcn-vue）提供 Provider 管理、模型映射配置、重试规则、请求日志查看、实时监控等功能。

## 分支策略

- `main` — 可发布分支，始终保持稳定可发布状态

**流程：** 功能分支 → PR 直接合并到 `main`（发布）

功能分支基于 `main` 创建，命名规范：`feat/xxx`、`fix/xxx`、`refactor/xxx`、`chore/xxx`

## 常用命令

```bash
# 后端开发（热重载，端口 9980）
npm run dev

# 后端构建 & 启动
npm run build
mkdir -p dist/db/migrations && cp src/db/migrations/*.sql dist/db/migrations/
FRONTEND_DIST=./frontend/dist npm start

# 前端开发（自动代理 /admin/api 到后端 :9980）
cd frontend && npm run dev

# 前端构建
cd frontend && npm run build

# 完整构建（tsc + 复制 migrations + 构建前端）
npm run build:full

# 测试
npm test                              # 全部测试
npx vitest run tests/auth.test.ts     # 单个测试文件
npm run test:watch                    # 监听模式

# Lint
npm run lint                          # ESLint（零警告容忍）

# Docker
docker compose up -d
```

## 架构

### 后端（Fastify + SQLite）

**入口层：**
- `src/cli.ts` — npm bin 入口（带 shebang），无条件调用 `main()`
- `src/index.ts` — 库入口，导出 `buildApp` 和 `main`。`buildApp()` 组装所有插件，支持注入 `db`（测试用 in-memory）
- `src/config.ts` — 单例配置，惰性缓存

**`buildApp()` 插件注册顺序：**
```
seedDefaultRules → ModelStateManager.init → RetryRuleMatcher.load
→ ProviderSemaphoreManager → RequestTracker → 初始化所有 provider 并发配置
→ authMiddleware → openaiProxy → anthropicProxy → adminRoutes → fastifyStatic
```

**代理层 `src/proxy/`（三层架构：Handler → Orchestrator → Transport）：**

| 文件 | 角色 |
|------|------|
| `proxy-handler.ts` | **Handler 层**：`handleProxyRequest()` — Fastify 路由回调，负责映射解析、header 构建、日志记录，调用 Orchestrator |
| `orchestrator.ts` | **Orchestrator 层**：`ProxyOrchestrator` — 协调信号量、tracker、resilience 三大 scope，驱动重试/failover 循环 |
| `resilience.ts` | 重试决策层（替代旧 `retry.ts`）：`ResilienceLayer` + fixed/exponential 策略，判断是否重试/failover |
| `transport.ts` | **Transport 层**：底层 HTTP 调用 `callNonStream()`/`callGet()`，构建原始 `http.request` |
| `stream-proxy.ts` | SSE 流式代理引擎：`StreamProxy` 类管理缓冲状态机 + `SSEMetricsTransform` 旁路采集 |
| `scope.ts` | 信号量/追踪器 scope 包装：`SemaphoreScope`（acquire/release）+ `TrackerScope`（start/complete） |
| `proxy-logging.ts` | 日志工具：header 脱敏、拦截日志、resilience 结果日志、transport 指标采集 |
| `log-helpers.ts` | DB 日志插入：`insertSuccessLog()` / `insertRejectedLog()`，携带 failover/retry 元数据 |
| `proxy-core.ts` | 共享工具：错误格式化工厂、上游 header 构建、GET 代理。Re-export `types.ts` 和 `transport.ts` 类型 |
| `types.ts` | 代理层常量和类型：`UPSTREAM_SUCCESS`、`RawHeaders`、`TransportResult`、`ProviderSwitchNeeded` |
| `openai.ts` | OpenAI 代理插件（`POST /v1/chat/completions`、`GET /v1/models`），注入 `stream_options` |
| `anthropic.ts` | Anthropic 代理插件（`POST /v1/messages`），与 openai.ts 对称 |
| `semaphore.ts` | Provider 级并发控制：基于 Promise 的等待队列，支持 AbortSignal 和超时 |
| `retry-rules.ts` | `RetryRuleMatcher`：从 DB 加载规则到内存，按 status_code 分组缓存 |
| `enhancement-handler.ts` | 代理增强：指令解析、命令拦截、会话记忆 |
| `directive-parser.ts` | 从 user 消息中提取 `$SELECT-MODEL` / `[router-model]` / `[router-command]` 标记 |
| `model-state.ts` | `ModelStateManager` 单例：内存 + SQLite 双层缓存，24h 滑动窗口 |
| `response-cleaner.ts` | 清理历史消息中的路由标签 |
| `mapping-resolver.ts` | 将 client_model 解析为 `{ backend_model, provider_id }` |
| `strategy/` | 四种路由策略：`scheduled`（定时）、`round-robin`（轮询）、`random`（随机）、`failover`（故障转移） |

**请求处理流程（三层调用链）：**
```
Handler (proxy-handler.ts)
  applyEnhancement → resolveMapping → buildHeaders
  → orchestrator.execute()
    → SemaphoreScope.acquire（队列满→503，超时→504）
    → ResilienceLayer（transportFn 循环：重试/failover 决策）
      → Transport (transport.ts / stream-proxy.ts)
    → TrackerScope.complete
  → insertSuccessLog + collectTransportMetrics
```

**认证 `src/middleware/`：**
- `auth.ts` — 全局 `onRequest` hook，Bearer token → SHA256 哈希 → 查 `router_keys` 表。跳过 `/health`、`/admin`
- `admin-auth.ts` — JWT + Cookie 认证。跳过 `/admin/api/setup/*`、`/admin/api/login`、`/admin/api/logout`

**数据库 `src/db/`（better-sqlite3）：**
- `index.ts` — `initDatabase()` 自动创建目录、执行 `src/db/migrations/*.sql`
- 按领域拆分文件：`providers.ts`、`mappings.ts`、`logs.ts`、`metrics.ts`、`stats.ts`、`retry-rules.ts`、`router-keys.ts`、`settings.ts`、`session-states.ts`、`helpers.ts`
- `helpers.ts` 提供 `buildUpdateQuery()`（白名单过滤安全字段的通用 UPDATE）和 `deleteById()`

**数据表（19 个迁移，11 张表）：**

| 表 | 核心用途 |
|----|---------|
| `providers` | 供应商（含并发控制字段：max_concurrency、queue_timeout_ms、max_queue_size） |
| `model_mappings` | 旧版单映射（保留兼容） |
| `mapping_groups` | 映射组（strategy: scheduled/round_robin/random/failover，rule 为 JSON） |
| `retry_rules` | 重试规则（status_code + body_pattern 正则 + fixed/exponential 策略） |
| `request_logs` | 请求日志（含完整链路：client_request/upstream_request/upstream_response/client_response） |
| `request_metrics` | Token 统计（input/output/cache、ttft、tps、stop_reason） |
| `router_keys` | 客户端密钥（SHA256 哈希存储 + AES 加密原文） |
| `settings` | 系统设置（密码哈希、加密密钥、JWT 密钥、proxy_enhancement） |
| `session_model_states` | 会话模型状态（router_key_id + session_id 联合唯一） |
| `session_model_history` | 会话模型变更历史 |

**监控层 `src/monitor/`：**
- `request-tracker.ts` — `RequestTracker`：活跃请求 Map + 最近完成列表（200 条/5min TTL）+ SSE 广播（6 种事件）
- `stats-aggregator.ts` — `StatsAggregator`：环形缓冲区（1000）存储延迟样本，计算 p50/p99
- `runtime-collector.ts` — `RuntimeCollector`：采集内存、句柄、事件循环延迟

**指标采集 `src/metrics/`：**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhushanwen321/llm-simple-router](https://github.com/zhushanwen321/llm-simple-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
