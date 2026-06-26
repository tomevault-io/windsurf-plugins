---
trigger: always_on
description: 面向本仓库的编码 agent 开发指南。目标是让后续开发少走弯路：先理解边界，再做小而准的改动，最后用合适的测试闭环。
---

# AGENTS.md

面向本仓库的编码 agent 开发指南。目标是让后续开发少走弯路：先理解边界，再做小而准的改动，最后用合适的测试闭环。

本文档基于实际代码阅读，描述的是当前代码库的真实架构和设计意图。

---

## 项目概述

本项目是 Go 编写的 **Twitter/X Media Downloader (TMD)**，支持 CLI、API Server 和内置 Web 管理界面。

- 语言：Go 1.25.0
- 数据库：SQLite（`modernc.org/sqlite`，纯 Go 实现，`CGO_ENABLED=0`）
- HTTP 客户端：`go-resty/resty/v2`
- 日志：`sirupsen/logrus` + `natefinch/lumberjack`（轮转） + `rifflock/lfshook`（文件 hook）
- 任务队列：自定义 `DownloadQueue`（基于 `sync.Cond`） + `ants/v2` goroutine 池
- 计划任务：自定义 `Scheduler`（基于 cron-like 解析）
- 前 端：原生 HTML/CSS/JS（无构建步骤）

---

## 一、整体架构分层

```
┌──────────────┐
│   main.go    │  ← 进程入口：配置、登录、数据库、分流
└──────┬───────┘
       │
       ├─ CLI 模式 ───────→ internal/cli
       │                     （同步执行，直接返回）
       │
       └─ Server 模式 ────→ internal/api
                            （异步任务，SSE 推送）
                            │
                            ├─ authMiddleware          ← Bearer Token 认证
                            ├─ Web UI (internal/api/web/)
                            ├─ TaskManager + DownloadQueue
                            ├─ EventBus → SSE 广播
                            └─ Scheduler（定时计划任务）
                                    │
                                    ▼
       ┌──────────────────────────────────────┐
       │        internal/service              │ ← 统一应用服务层
       │        DownloadService 接口           │
       │        11 种下载/操作方法的统一入口     │
       └──────────────────┬───────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
  internal/downloading  internal/twitter  internal/database
  （业务编排）          （Twitter API）    （SQLite 持久化）
          │
          ▼
  internal/downloader ← internal/entity
  （单文件下载）        （实体抽象）
          │
          ▼
  internal/naming （命名规则）
  internal/path   （路径管理）
  internal/utils  （通用工具）

  其他支撑层：
  internal/config      ← 配置管理（YAML + 环境变量）
  internal/consolelog  ← 控制台日志捕获（供 Web UI 实时显示）
  internal/scheduler   ← 计划任务系统
```

---

## 二、进程入口 — `main.go`

`main.go` 是唯一的进程入口。按严格顺序执行以下步骤：

### 2.1 全局预解析 (`parseBootstrapArgs`)
只识别**引导级参数**，不解析下载参数：
- `-server` → 启动 Server 模式
- `-port <n>` → 指定 HTTP 端口（默认 25556，支持 `$TMD_PORT` 环境变量）
- `-dbg` → 调试模式
- `-conf` → 交互式配置模式

其余参数原样保留为 `cliArgs` 传递给 CLI 模式。

### 2.2 初始化顺序
1. **确定应用根目录** (`resolveAppRootPath`)：优先 `$TMD_HOME`，其次 `$APPDATA/.tmd2`（Windows）或 `$HOME/.tmd2`
2. **初始化日志**：logrus + lumberjack 文件轮转 + consolelog 捕获
3. **加载配置**：`conf.yaml` + `TMD_*` 环境变量覆盖
4. **代理设置**：`conf.yaml` 中的 `proxy_url` 覆盖系统设置；无代理时自动同步 HTTP_PROXY ↔ HTTPS_PROXY
5. **分流**：`-server` 标志 → `runServer()`，否则 → `cli.Execute()`

### 2.3 两种模式的差异

| 方面 | CLI 模式 | Server 模式 |
|------|----------|-------------|
| 执行方式 | 同步，等待完成后退 | 异步，创建任务后立即返回 |
| 客户端日志 | O_TRUNC（每次截断重新写入） | O_APPEND（追加模式） |
| 信号处理 | `cancel()`（取消 context） | `server.GracefulShutdown()` |
| 资源清理 | `defer db.Close()` | `GracefulShutdown` 统一处理 |
| 进度报告 | `LogReporter`（日志输出） | `SSEProgressReporter`（推送至 Web UI） |

### 2.4 数据目录结构

```
{appRootPath}（默认 ~/.tmd2 或 %APPDATA%\.tmd2）
├── conf.yaml                  ← 主配置
├── additional_cookies.yaml    ← 额外账号 cookies
├── tmd2.log                   ← 程序日志（轮转）
├── client.log                 ← HTTP 客户端日志
└── schedules.yaml             ← 计划任务配置

{rootPath}（下载根目录，由配置 root_path 指定）
├── users/                     ← 用户下载目录
│   ├── {screen_name}/         ← 每个用户一个目录
│   │   ├── ... 媒体文件
│   │   └── .loongtweet/       ← 推文元数据 JSON/TXT
│   ├── {list_name}/           ← List 目录（通过软链接关联用户）
│   └── ...
└── .data/
    ├── foo.db                 ← SQLite 数据库
    ├── errors.json            ← 常规下载失败记录
    └── json_errors.json       ← JSON 导入下载失败记录
```

---

## 三、两条主调用链

### 3.1 CLI 模式调用链

```text
main.go
  → cli.Execute(ctx, args, deps)
    → cli.ParseArgs(args)           # args.go：解析 -user / -list / -foll / -jsonfile 等
    → cliTaskSelection.primaryMode() # 判断首要任务类型，处理优先级
    → switch on primaryMode:
        case cliTaskModeJSONFile:
          → service.JsonFileDownload(ctx, "cli", paths, noRetry, reporter)
        case cliTaskModeJSONFolder:
          → service.JsonFolderDownload(ctx, "cli", paths, noRetry, reporter)
        case cliTaskModeMarkDownloaded:
          → service.MarkDownloaded(ctx, "cli", users, lists, following, markTime, reporter)
        case cliTaskModeBatch:
          → 单用户: service.UserDownload()
            单关注: service.FollowingDownload()
            多/混合: service.BatchDownload()
        case cliTaskModeProfile:
          → service.ProfileDownload() / service.ListProfileDownload()
```

**关键设计**：
- CLI 参数有严格优先级：`-jsonfile` > `-jsonfolder` > `-mark-downloaded` > `-user/-list/-foll`（可组合）> `-profile-user/-profile-list`（可组合）
- 高优先级参数独占执行，低优先级被忽略
- 批量下载与 Profile 下载可以同时执行

### 3.2 API Server 模式调用链

```text
HTTP 请求
  → [authMiddleware] Bearer Token 认证（可选）   ← 新增认证层
  → [CORS] 跨域处理
  → server.buildHandler() 路由分发
    → handler 函数（如 handleUserDownload）
      → taskManager.CreateTask(type, data)    创建任务 → task_id
      → buildTaskRunFunc(task)                构建执行闭包
      → downloadQueue.Enqueue(task, runFunc)   入队
      → 返回 HTTP 202 (Accepted) + task_id

后台：
  → DownloadQueue.workerLoop()
    → nextJob() → 持有 target 锁（防止同名用户并发下载）
    → UpdateTaskStatus(id, TaskStatusRunning)
    → SSEProgressReporter.OnProgress()
    → goroutine 执行 runFunc(task.Ctx, taskID, reporter)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leeexx2001/tmd](https://github.com/leeexx2001/tmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
