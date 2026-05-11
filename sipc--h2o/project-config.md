---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

H2O 是企业内网使用的 Hysteria2 订阅与节点认证管理面板：后端提供用户/套餐/订阅/节点 CRUD、Hysteria2 节点 HTTP 认证回调、Hysteria 订阅链接生成；前端提供管理员后台与普通用户自助面板。Next.js 16 App Router + React 19 + TypeScript + Tailwind v4 + shadcn/ui。

## 常用命令

使用 **pnpm**（仓库有 `pnpm-lock.yaml`）。

- `pnpm dev` — 启动 Turbopack 开发服务器（`next dev --turbopack`）
- `pnpm build` / `pnpm start` — 生产构建与启动
- `pnpm lint` — ESLint（基于 `eslint-config-next`，包含 core-web-vitals + typescript）
- `pnpm typecheck` — `tsc --noEmit`，增量编译结果在 `tsconfig.tsbuildinfo`
- `pnpm format` — Prettier + tailwind 插件（`no-semi`、双引号、2 空格、LF、`tailwindFunctions: ["cn","cva"]`）
- 新增 shadcn 组件：`npx shadcn@latest add <name>`（`components.json` 里 style 是 `radix-nova`，基色 `neutral`，图标库 `lucide`）

目前仓库没有测试脚手架。

## 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `H2O_DB_PATH` | `./data/h2o.sqlite` | 业务数据库路径 |
| `H2O_LOGS_DB_PATH` | `./data/h2o-logs.sqlite` | 日志数据库路径 |
| `H2O_SECURE_COOKIE` | `"true"`（生产） | 设为 `"false"` 可在纯 HTTP 部署下关闭 cookie Secure 标志 |

## 首次启动流程

1. Turnstile 人机验证通过后台「站点设置」配置（可选）：
   - 两个 key 都缺失 → 人机验证视为 **disabled**，前端不渲染 widget
   - 两个 key 都有 → **enabled**
   - 只配一个 → **misconfigured**，登录/注册会直接报错（见 `lib/turnstile.ts`）
2. 启动后访问 `/init` 引导创建第一个管理员（`POST /api/auth/bootstrap-admin`）。已存在 admin 时该接口返回 `ADMIN_EXISTS`。
3. 数据库首次调用 `getDb()` / `getLogsDb()` 时自动建表，无外部迁移工具。

## 架构要点

### 双 SQLite 文件（Node 内建驱动）

使用 Node.js 内建 `node:sqlite` 的 `DatabaseSync`，**没有** `better-sqlite3`/`sqlite3` 依赖。需要 Node 版本支持 `node:sqlite`（Node 22+ 实验性 / Node 23+ 稳定）。

- `lib/db.ts` → 业务库 `data/h2o.sqlite`（可用 `H2O_DB_PATH` 覆盖）：`users`, `nodes`, `plans`, `plan_nodes`, `subscriptions`, `sessions`, `settings`, `node_stats`, `node_user_traffic`, `traffic_hourly_stats`, `node_hourly_traffic`, `subscription_hourly_traffic`
- `lib/logs-db.ts` → 日志库 `data/h2o-logs.sqlite`（可用 `H2O_LOGS_DB_PATH` 覆盖）：`auth_logs`（节点认证日志）、`event_logs`（业务事件日志）

两库分离的设计目的是让日志可单独归档/清理，不影响业务库。两者都采用单例 + 懒加载，首次取用时 `migrate()` 建表并打开 `PRAGMA foreign_keys = ON`。`migrate()` 只维护当前 schema，**不做旧版本迁移兼容**。

#### 完整数据库 Schema

**业务库 `h2o.sqlite`**

`users`

| 列 | 类型 | 约束 |
|----|------|------|
| `id` | `INTEGER` | `PRIMARY KEY AUTOINCREMENT` |
| `username` | `TEXT` | `NOT NULL UNIQUE` |
| `password_hash` | `TEXT` | `NOT NULL`（格式 `scrypt$salt$hash`） |
| `auth_token` | `TEXT` | `NOT NULL UNIQUE`（24 字节随机 hex，48 字符） |
| `role` | `TEXT` | `NOT NULL DEFAULT 'user'`, `CHECK(role IN ('user','admin'))` |
| `status` | `TEXT` | `NOT NULL DEFAULT 'active'`, `CHECK(status IN ('active','disabled'))` |
| `created_at` | `TEXT` | `NOT NULL DEFAULT (datetime('now'))` |
| `updated_at` | `TEXT` | `NOT NULL DEFAULT (datetime('now'))` |
| `last_login_at` | `TEXT` | nullable |

`nodes`

| 列 | 类型 | 约束 |
|----|------|------|
| `id` | `INTEGER` | `PRIMARY KEY AUTOINCREMENT` |
| `name` | `TEXT` | `NOT NULL UNIQUE` |
| `ip` | `TEXT` | `NOT NULL` |
| `port` | `INTEGER` | `NOT NULL` |
| `port_hopping` | `TEXT` | nullable（逗号分隔端口范围，如 `443,5000-6000`） |
| `auth_path` | `TEXT` | `NOT NULL UNIQUE`（节点认证路径，也用作 agent 密钥） |
| `status` | `TEXT` | `NOT NULL DEFAULT 'enabled'`, `CHECK(status IN ('enabled','disabled'))` |
| `sni` | `TEXT` | nullable |
| `obfs` | `TEXT` | nullable |
| `obfs_password` | `TEXT` | nullable |
| `insecure` | `INTEGER` | `NOT NULL DEFAULT 0`, `CHECK(insecure IN (0,1))` |
| `pin_sha256` | `TEXT` | nullable |
| `node_ip` | `TEXT` | nullable（agent 上报的实际 IP） |
| `node_port` | `INTEGER` | nullable（agent 上报的实际端口） |
| `node_port_hopping` | `TEXT` | nullable |
| `cert_mode` | `TEXT` | `NOT NULL DEFAULT 'self-signed'`（`self-signed` / `file` / `acme`） |
| `cert_path` | `TEXT` | nullable |
| `key_path` | `TEXT` | nullable |
| `acme_domains` | `TEXT` | nullable |
| `acme_email` | `TEXT` | nullable |
| `acme_dns_provider` | `TEXT` | nullable |
| `acme_dns_config` | `TEXT` | nullable |
| `masquerade_type` | `TEXT` | nullable |
| `masquerade_config` | `TEXT` | nullable |
| `agent_interval` | `INTEGER` | nullable（agent 上报间隔秒数） |
| `created_at` | `TEXT` | `NOT NULL DEFAULT (datetime('now'))` |

`plans`

| 列 | 类型 | 约束 |
|----|------|------|
| `id` | `INTEGER` | `PRIMARY KEY AUTOINCREMENT` |
| `name` | `TEXT` | `NOT NULL UNIQUE` |
| `traffic_limit_bytes` | `INTEGER` | `NOT NULL` |
| `duration_days` | `INTEGER` | `NOT NULL` |
| `up_mbps` | `INTEGER` | `NOT NULL DEFAULT 0`（0 = 不限速） |
| `down_mbps` | `INTEGER` | `NOT NULL DEFAULT 0`（0 = 不限速） |

`plan_nodes`（多对多关联表）

| 列 | 类型 | 约束 |
|----|------|------|
| `plan_id` | `INTEGER` | `NOT NULL`, `FK → plans(id) ON DELETE CASCADE` |
| `node_id` | `INTEGER` | `NOT NULL`, `FK → nodes(id) ON DELETE CASCADE` |
| | | `PRIMARY KEY (plan_id, node_id)` |

`subscriptions`

| 列 | 类型 | 约束 |
|----|------|------|
| `id` | `INTEGER` | `PRIMARY KEY AUTOINCREMENT` |
| `user_id` | `INTEGER` | `NOT NULL`, `FK → users(id) ON DELETE CASCADE` |
| `plan_id` | `INTEGER` | `NOT NULL`, `FK → plans(id)` |
| `start_time` | `TEXT` | `NOT NULL` |
| `expire_time` | `TEXT` | `NOT NULL` |
| `used_traffic_bytes` | `INTEGER` | `NOT NULL DEFAULT 0` |
| `status` | `TEXT` | `NOT NULL DEFAULT 'active'`, `CHECK(status IN ('active','expired','blocked'))` |

`sessions`

| 列 | 类型 | 约束 |
|----|------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SIPC/H2O](https://github.com/SIPC/H2O) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
