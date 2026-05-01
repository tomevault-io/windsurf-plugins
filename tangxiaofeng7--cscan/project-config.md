---
trigger: always_on
description: **cscan** — 企业级分布式网络资产扫描平台
---

# CLAUDE.md
## 一、项目架构

### 1.1 项目概述

**cscan** — 企业级分布式网络资产扫描平台

### 1.2 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 后端框架 | Go + go-zero 微服务框架 | Go 1.25+ / go-zero v1.7.3 |
| 前端框架 | Vue 3 + Vite + Element Plus | Vue 3.4 / Vite 5 / Element Plus 2.4 |
| 数据库 | MongoDB + Redis | MongoDB 6 / Redis 7 |
| RPC 通信 | gRPC + Protobuf | grpc v1.76 |
| 扫描引擎 | ProjectDiscovery + Nmap/Masscan | nuclei v3.6 / httpx v1.7 / naabu v2.3 / subfinder v2.11 |
| 截图引擎 | Chromedp (Chrome 无头浏览器) | chromedp v0.14 |
| 任务调度 | robfig/cron + Redis Sorted Set | cron/v3 |
| 认证 | JWT (golang-jwt/v4) | jwt/v4 v4.5 |
| 前端状态管理 | Pinia | v2.1 |
| 国际化 | vue-i18n | v11 |
| 图表 | ECharts | v5.4 |
| CSS 预处理 | SCSS (modern-compiler API) | sass v1.97 |
| 测试 (Go) | testify + gopter (属性测试) | testify v1.11 / gopter v0.2 |
| 测试 (前端) | Vitest + happy-dom + fast-check | vitest v4 |

### 1.3 系统架构图

```
[Browser] → [Vue 3 Frontend (web/) :3000]
                  │
            [Vite Proxy /api → :8888]
                  │
           [HTTP API (api/) - go-zero REST :8888]
                  │
      ┌───────────┼────────────┐
      │           │            │
[gRPC RPC     [MongoDB]    [Redis]
 (rpc/) :9000]    │            │
                  │      ┌─────┴──────────┐
                  │  [Sorted Set     [Pub/Sub
                  │   任务队列]     cscan:cron:execute]
                  │      │               │
              [Scheduler (scheduler/)]───┘
                  │
         [Worker nodes (worker/)]  ← 通过 Install Key 认证，WebSocket 长连接
                  │
         [Scanner modules (scanner/)]
                  │
    ┌─────┬──────┼──────┬────────┬──────┐
  Naabu  Nmap  Httpx  Subfinder Nuclei Chromedp
```

### 1.4 核心架构模式

- **多租户隔离**: 所有数据按 `workspace_id` 过滤；MongoDB 集合命名 `{workspaceId}_{entity}`（如 `default_asset`、`ws1_tasks`）
- **任务流**: MainTask → 按目标数量分批 (batchSize=50) → SubTasks → 推入 Redis Sorted Set 队列 → Worker 拉取执行
- **孤儿恢复**: 后台协程每 5 分钟检查 STARTED 状态超过 30 分钟未更新的任务，重置为 PENDING 并重新入队
- **定时调度**: Redis Pub/Sub 频道 `cscan:cron:execute` 触发定时扫描，基于 `robfig/cron/v3`
- **Worker 通信**: Install Key 认证注册 → WebSocket (`/api/v1/worker/ws`) 保持长连接 → 心跳上报 → REST 接口回传结果

---

## 二、项目模块划分

### 2.1 文件与文件夹布局

```
cscan/
├── api/                          # HTTP API 服务（主入口）
│   ├── cscan.go                  # API 服务入口点
│   ├── etc/cscan.yaml            # API 配置（端口8888, JWT, MongoDB, Redis, RPC）
│   └── internal/
│       ├── config/config.go      # 配置结构体定义
│       ├── handler/              # 路由处理器（按资源域分子包）
│       │   ├── routes.go         # 统一路由注册（4 层认证级别）
│       │   ├── asset/            # 资产管理 Handler
│       │   ├── task/             # 任务管理 Handler
│       │   ├── vul/              # 漏洞管理 Handler
│       │   ├── worker/           # Worker 管理 Handler
│       │   ├── fingerprint/      # 指纹管理 Handler
│       │   ├── poc/              # POC 管理 Handler
│       │   ├── onlineapi/        # 在线 API 搜索 Handler
│       │   ├── user/             # 用户管理 Handler
│       │   ├── workspace/        # 工作空间 Handler
│       │   ├── organization/     # 组织管理 Handler
│       │   ├── blacklist/        # 黑名单 Handler
│       │   ├── dirscan/          # 目录扫描 Handler
│       │   ├── subdomain/        # 子域名字典 Handler
│       │   ├── subfinder/        # Subfinder 配置 Handler
│       │   ├── notify/           # 通知配置 Handler
│       │   ├── report/           # 报告 Handler
│       │   └── ai/               # AI POC 生成 Handler
│       ├── logic/                # 业务逻辑层（平铺，{动作}{实体}logic.go）
│       ├── middleware/           # 中间件（JWT/WorkerAuth/ConsoleAuth）
│       ├── svc/                  # 服务上下文（DI 容器 + 服务实现）
│       │   ├── servicecontext.go # ServiceContext 核心结构体
│       │   ├── scanresult_service.go
│       │   ├── history_service.go
│       │   └── sync/             # 同步服务（模板/指纹/POC 同步）
│       └── types/                # 请求/响应类型定义
├── rpc/                          # gRPC 内部服务
│   └── task/
│       ├── task.go               # RPC 服务入口（端口 9000）
│       ├── task.proto            # Protobuf 定义
│       ├── pb/                   # 生成的 pb 代码
│       ├── taskservice/          # 服务实现
│       ├── client/               # 客户端封装
│       └── etc/task.yaml         # RPC 配置
├── model/                        # MongoDB 数据模型（30+ 模型文件）
│   ├── asset.go                  # 资产模型（按 workspace 分集合）
│   ├── task.go                   # 主任务/子任务模型
│   ├── vul.go                    # 漏洞模型
│   ├── user.go                   # 用户模型（全局集合）
│   ├── workspace.go              # 工作空间模型
│   ├── fingerprint.go            # 指纹模型
│   ├── scantemplate.go           # 扫描模板
│   ├── indexes.go                # MongoDB 索引定义
│   ├── base.go                   # 基础类型
│   └── errors.go                 # 模型层错误定义
├── pkg/                          # 共享工具包
│   ├── xerr/                     # 业务错误码体系
│   │   ├── errcode.go            # 错误码常量 + 消息映射
│   │   ├── errors.go             # CodeError 结构体 + 工厂函数
│   │   └── scan_errors.go        # 扫描相关错误
│   ├── response/response.go      # 统一 HTTP 响应封装
│   ├── cache/                    # 缓存工具
│   ├── circuitbreaker/           # 熔断器
│   ├── httpclient/               # HTTP 客户端封装
│   ├── notify/                   # 多渠道通知发送
│   ├── retry/                    # 重试机制
│   ├── risk/                     # 风险等级计算
│   └── utils/                    # 通用工具函数
├── scheduler/                    # 任务调度器（Redis 队列 + Cron）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tangxiaofeng7/cscan](https://github.com/tangxiaofeng7/cscan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
