---
trigger: always_on
description: **最后更新**: 2026-06-16 · **分支**: main · **版本**: v1.29.1 · **提交**: aa9900d2
---

# HotPlex 项目知识库

**最后更新**: 2026-06-16 · **分支**: main · **版本**: v1.29.1 · **提交**: aa9900d2

---

## 目录

- [约定与规范](#约定与规范)
- [项目结构](#项目结构)
- [开发指南](#开发指南)（含 [PR Review 修复循环](#pr-review-修复循环)）
- [配置参考](#配置参考)
- [命令参考](#命令参考)
- [附录](#附录)

---

## 约定与规范

### 必须遵守

- **Mutex**: 显式 `mu` 字段，不嵌入，不传指针
- **错误**: `Err` 前缀（哨兵）、`Error` 后缀（自定义）、`fmt.Errorf("%w")` 包装
- **日志**: `log/slog` JSON handler
- **测试**: `testify/require`、table-driven、`t.Parallel()`、单模块 ≤5s（`-count=1 -race`）、禁止 `time.Sleep` 等待异步结果（改用 `require.Eventually` 或 channel 信号）
- **Worker 注册**: `init()` + `worker.Register()` 模式
- **关闭顺序**: signal → cancel ctx → tracing → hub → bridge → sessionMgr → HTTP
- **服务重启**: 必须使用 `hotplex service restart` 原子指令，禁止手动拆分 `stop && sleep && start`（仅二进制替换场景需手动 stop 等待）
- **非 main 分支 push**: 非 main 分支本地验证通过后直接 commit + push，无需询问用户确认
- **Git Hooks**: clone 后必须执行 `make hooks` 安装 hooks（pre-commit 跑 gofmt + golangci-lint，pre-push 跑完整质量门禁），禁止跳过

### 反模式（禁止）

- ❌ `sync.Mutex` 嵌入或传指针
- ❌ `math/rand` 用于加密
- ❌ Shell 执行（仅允许 `claude` 二进制）
- ❌ 硬编码路径分隔符
- ❌ 直接使用 POSIX 信号
- ❌ 用 `sed`/`awk` 插入或修改源码行（缩进不可控，必须用 Edit 工具）

### 代码编辑规则

- **Edit 工具优先**：修改源码必须使用 Edit 工具，禁止用 `sed -i` 插入或修改代码行
- **Edit 匹配失败时**：重新 Read 文件获取精确内容，用精确字符串重试 Edit；扩大上下文使其唯一
- **Go 文件 tab 缩进**：Go 项目使用 tab 缩进（gofmt 标准）。使用 Edit 工具时，old_string 必须从 Read 输出中直接复制原文（保留 tab），禁止手敲空格缩进。Edit 匹配失败时先用 `cat -A` 确认实际空白字符
- **`sed` 适用场景**：仅限非代码操作（config 快速替换、日志过滤等简单唯一 token 替换）

### 独特风格

- **锁顺序**: `m.mu` → `ms.mu`（防止死锁）
- **背压**: 丢弃 `message.delta`，保留 `state`/`done`/`error`
- **Seq 分配**: Per-session 原子单调计数器
- **进程终止**: 3 层（SIGTERM → 等待 5s → SIGKILL）
- **Detached Restart**: `--detached` fork 独立 PGID helper，60s 冷却期防循环（`pid.go: restartMarker`）
- **Agent 配置**: B/C 双通道注入（通道结构、加载方式、fallback 规则见 [配置参考](#配置参考)）
- **元认知层**: `internal/agentconfig/META-COGNITION.md` 定义 Worker 的身份边界（不管理 Transport/状态/协议）、B/C 通道冲突隔离法则（directives 无条件覆盖 context）、配置替换的"命中即终止"机制、配置修改 SOP（禁止改全局来影响 Bot）
- **XML 安全**: 强制开启 **XML Sanitizer**，对保留标签进行 HTML 转义预防注入
- **Windows 注入**: 强制使用 **临时文件注入**（`--append-system-prompt-file`），严禁使用内联参数防止 cmd.exe 截断

---

## 项目结构

### 入口点 (`cmd/hotplex/`)

| 文件                                       | 功能                                                                   |
| ------------------------------------------ | ---------------------------------------------------------------------- |
| `main.go`                                  | CLI Root (cobra 根命令)                                                |
| `gateway_run.go`                           | GatewayDeps (DI 容器、信号处理、hub/session/bridge 初始化)             |
| `gateway_cmd.go`                           | gateway 子命令：start/stop/restart + `--detached`                      |
| `gateway_restart_helper.go`                | Restart Helper（独立 PGID，Worker-initiated restart）                  |
| `gateway_restart_helper_{unix,windows}.go` | 平台隔离（Setpgid / CREATE_NEW_PROCESS_GROUP）                         |
| `routes.go`                                | HTTP 路由注册                                                          |
| `messaging_init.go`                        | 消息适配器生命周期（多 bot 初始化 + fillSlackExtras/fillFeishuExtras） |
| `cron_cmd.go`                              | cron 子命令注册                                                        |
| `cron_*`                                   | cron CRUD CLI（create/update/delete/get/list/trigger/history）         |
| `service_*.go`                             | 系统服务管理（systemd/launchd/SCM）                                    |
| `update.go`                                | 自更新命令：GitHub API、下载、校验、替换                               |

### 核心模块 (`internal/`)

**Gateway** (`internal/gateway/`)：
- `hub.go` - `Hub` WS 广播 hub（`routeMessage`、`removeSession`、`SessionWriter` 接口）
- `conn.go` - `Conn` 单个 WS 连接 (ReadPump/WritePump、performInit 四阶段 dispatch)
- `handler.go` - `Handler` AEP 事件分发（`SessionManager` 子接口组合：SessionReader/Lifecycle/Transitioner/WorkerManager/Admin）
- `bridge.go` - `Bridge` Session ↔ Worker 生命周期编排
- `bridge_forward.go` - `forwardEvents` 事件转发循环（`forwardContext` 单 goroutine 所有权）
- `llm_retry.go` - `LLMRetryController` 自动重试
- `api.go` - `GatewayAPI` HTTP session 端点
- `worker_cmds.go` - Worker 命令分发（context usage/MCP status/set model 等）
- `platform_writer.go` - `RouteWrite` 按 WS conn 编码写入（替代 pre-encoded 广播）

**Session** (`internal/session/`)：
- `manager.go` - `Manager` 5 状态机、状态迁移、GC
- `store.go` - SQLite 持久化
- `pg_store.go` - PostgreSQL 持久化
- `key.go` - `DeriveSessionKey` UUIDv5 确定性 session ID
- `pool.go` - `PoolManager` 全局 + 每用户配额

**Messaging** (`internal/messaging/`)：
- `bridge.go` - `Bridge` StartSession → Join → Handle
- `platform_adapter.go` - `PlatformAdapter` 基座（共享状态、ConfigureWith）
- `platform_types.go` - `PlatformType` 常量 + `ExtractPlatformKeys`
- `platform_interfaces.go` - `HubInterface`、`HandlerInterface`、`SessionStarter`、`PlatformAdapterInterface`
- `platform_registry.go` - 适配器注册表（Register/New/RegisteredTypes）
- `control_command.go` - `ParseControlCommand` 斜杠命令解析
- `bot_registry.go` - `BotRegistry` 并发安全多 bot 注册表
- `config.go` - `AdapterConfig` 含 `BotName` 字段
- `slack/` - Socket Mode 适配器
- `feishu/` - 飞书 WS 适配器 + STT + 卡片模板
- `yuanxin/` - 元芯平台适配器
- `tts/` - Edge-TTS 语音合成 + FFmpeg Opus 转换
- `stt/` - 语音转文字（独立 STT 模块）
- `toolfmt/` - 工具调用格式化
- `phrases/` - 短语模板
- `mock/` - 测试 mock

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hrygo/hotplex](https://github.com/hrygo/hotplex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
