---
trigger: always_on
description: `CPA-Core-LTS` 是 `router-for-me/CLIProxyAPI` 的长期维护分支：跟踪 upstream latest，同时稳定保留 `v6.9.49` 基线已有的完整 usage statistics、`CPA-Panel-LTS` 兼容性，以及本仓库 downstream 专属修改。
---

# CPA-Core-LTS agent instructions

## Purpose

`CPA-Core-LTS` 是 `router-for-me/CLIProxyAPI` 的长期维护分支：跟踪 upstream latest，同时稳定保留 `v6.9.49` 基线已有的完整 usage statistics、`CPA-Panel-LTS` 兼容性，以及本仓库 downstream 专属修改。

本仓库不是普通同步 fork。任何维护动作都必须先判断是否影响 LTS 统计契约、Management API、auth/config 兼容性、runtime usage attribution 和配套面板。

## Codex startup behavior

- Codex 通常从仓库根目录启动；本文件是启动期主规则和目录 router。
- 子目录 `AGENTS.md` 是按需 navigation card。从根目录启动时，它们通常不会自动进入上下文。
- 修改带有本地 `AGENTS.md` 的目录前，先运行 `cat <path>/AGENTS.md` 读取对应卡片。
- 如果目标路径上有多层 `AGENTS.md`，按从浅到深的顺序读取；冲突时更深层规则优先。
- `.github/workflows/agents-md-guard.yml` 会限制修改 `AGENTS.md` 的 PR：OWNER 可直接放行，MEMBER/COLLABORATOR 需要 `allow-agents-md-update` label，外部 PR 触碰 `AGENTS.md` 会被关闭。除非用户明确要求维护 agent 指令，不要把 AGENTS 改动混入产品代码 PR。

## LTS contract

- LTS 仓库：`https://github.com/BlueSkyXN/CPA-Core-LTS`
- 上游来源：`https://github.com/router-for-me/CLIProxyAPI`
- 基线版本 / 提交：`v6.9.49` / `b8bba053fcdafd80abc2152c88c78f4e7713c05a`
- 配套面板：`https://github.com/BlueSkyXN/CPA-Panel-LTS`
- Go module path 当前跟随 upstream major path：`github.com/router-for-me/CLIProxyAPI/v7`；不要因为 LTS 仓库名而随意改 import path。

必须保留：`usage-statistics-enabled`、`internal/usage/`、`/v0/management/usage`、`/v0/management/usage/export`、`/v0/management/usage/import`、`/v0/management/usage-queue`、`/v0/management/usage-statistics-enabled`，以及 API key、auth file、model、token、latency、success/failure、auth index 等统计字段。Core 默认从 `BlueSkyXN/CPA-Panel-LTS` latest release 下载 `management.html`，并保持 Panel `/usage` 页面、provider status bar、request events table 兼容。

## Protected full-sync workflow

本仓库使用人工 / AI 操作的 protected full-sync，不安排自动同步任务。`main` 是唯一 LTS 主线；`upstream/main` 只是只读同步坐标。真实 upstream sync 前必须阅读 `docs/lts/sync-runbook.md`。

同步原则：从最新 `origin/main` 创建隔离 worktree / 分支；fetch 后按 upstream first-parent SHA 分段；使用 `git merge --no-ff --log <UPSTREAM_STAGE_SHA>` 合入上游历史；普通 provider/model/translator/runtime/security/crash/stream 修复默认吸收；冲突触碰 protected deltas 时保留或重放 CPA-Core-LTS 行为。sync PR body 必须写 upstream from/to SHA、stage、冲突文件、protected delta review、contract/build/test 状态和覆盖的旧 upstream-port PR。合入 `main` 必须使用 Create a merge commit，禁止 squash 或 rebase sync PR。

如果 upstream diff 触碰 request lifecycle、auth identity、model resolution、token accounting、logging metadata、Management usage response shape、config hot reload、panel release source、plugin runtime 或 usage queue，即使没有文本冲突，也必须写 `Protected delta review`。

## Directory map

| Path | Responsibility | Local AGENTS.md | Read when |
|---|---|---:|---|
| `.github/` | Actions、PR guard、release、path/contract checks | Yes | 修改 workflow、权限、release、PR guard、CI gate 前 |
| `.codex/` | 本地 Codex 配置/临时上下文 | No | 仅当用户明确要求维护本地 Codex 资产 |
| `.gocache/` | 本地 Go build/test cache | No | 默认忽略，不纳入产品改动或验收结论 |
| `.playwright-mcp/` | 本地 Playwright MCP 运行残留/配置 | No | 默认忽略，不纳入产品改动 |
| `assets/` | README 展示图片和赞助资产 | No | 修改 README 图片引用或展示资产前 |
| `auths/` | auth 目录占位；运行时挂载真实凭据目录 | No | 默认不要提交真实 token/auth file；形状变更读 `internal/auth/` 和 `internal/watcher/` |
| `cmd/server/` | 服务端入口、CLI flags、TUI/standalone/home 启动 | Yes | 修改启动参数、登录流程入口、build metadata、server mode 前 |
| `cmd/fetch_antigravity_models/` | Antigravity model catalog 辅助拉取命令 | No | 修改模型拉取辅助工具前，同时读 `internal/auth/`、`internal/registry/` 卡片 |
| `cmd/fetch_codex_models/` | Codex model catalog 辅助拉取命令 | No | 修改 Codex 模型拉取、token refresh、client_version 或输出格式前 |
| `config.example.yaml` | 用户配置示例和 schema 可见面 | No | 新增/改名/删除 config key 前，同时读 `internal/config/AGENTS.md` |
| `Dockerfile` / `docker-compose*.yml` / `docker-build.*` | 容器构建和本地 Docker 运行 | No | 修改镜像、端口、volume、usage backup 脚本前 |
| `docs/` | SDK 文档、多语言 README 相关资料 | No | 修改 SDK 行为或公开接口文档前 |
| `docs/lts/` | LTS contract registry、protected delta、sync runbook | Yes | 修改 protected contract、sync runbook、guard marker 前 |
| `examples/` | SDK/translator/plugin/http-request 示例 | No | 修改公开示例或 API 使用方式前，必要时检查对应 SDK 卡片 |
| `internal/access/` | API key/access manager 适配 | No | 修改鉴权判定或 auth manager 集成前，同时读 `internal/auth/AGENTS.md` |
| `internal/api/` | Gin server、middleware、Management API、Amp/WebSocket endpoints | Yes | 修改 routes、middleware、Management API、Amp endpoints、HTTP/WebSocket 协议前 |
| `internal/auth/` | OAuth/device auth、token storage、credential helpers | Yes | 修改 token、OAuth callback、auth file、credential 保存/刷新前 |
| `internal/cache/` | Signature/cache helpers | No | 修改 Antigravity/Claude signature cache 前，同时读 translator/executor 卡片 |
| `internal/cmd/` | CLI login/import command helpers | No | 修改登录命令、vertex import、auth manager CLI 前 |
| `internal/config/` | YAML config model、defaults、sanitize、compat helpers | Yes | 新增/迁移/删除 config key 或改变默认值前 |
| `internal/logging/` | request/global logging、request metadata、log rotation | Yes | 修改日志格式、落盘策略、request metadata、redaction 前 |
| `internal/managementasset/` | `management.html` 下载和更新 | Yes | 修改面板下载源、缓存路径、auto-update 行为前 |
| `internal/pluginhost/` | Dynamic plugin ABI host、callbacks、scheduler、management routes | Yes | 修改 plugin lifecycle、callbacks、ABI/RPC、scheduler、stream bridge 前 |
| `internal/pluginstore/` | Plugin registry lookup、GitHub release install、checksum/install logic | Yes | 修改 plugin store registry、download、checksum、install/update 前 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlueSkyXN/CPA-Core-LTS](https://github.com/BlueSkyXN/CPA-Core-LTS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
