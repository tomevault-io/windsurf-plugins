---
trigger: always_on
description: - Monorepo with pnpm workspaces
---

# OpenFinClaw CLI — Repository Guidelines

## Project Structure

- Monorepo with pnpm workspaces
- `packages/core/` — `@openfinclaw/core`: Platform-independent business logic (API clients, types, schemas). Zero framework dependencies.
- `packages/cli/` — `@openfinclaw/cli`: MCP Server + CLI + interactive init wizard. Depends on `@modelcontextprotocol/sdk` and `@clack/prompts`.
- `configs/` — Example MCP configuration files for various agent platforms.

## Build & Development

- Runtime: Node 18+
- Install: `pnpm install`
- Build: `pnpm build` (runs `tsc` in both packages, core first)
- Core `tsconfig.json` excludes `src/**/__tests__/**` so tests are not emitted into `dist/` (smaller npm tarball); Vitest still runs `*.test.ts` from source.
- Run CLI: `OPENFINCLAW_API_KEY=<fch_...> node packages/cli/dist/index.js <command>`
- Run MCP: `OPENFINCLAW_API_KEY=<fch_...> node packages/cli/dist/index.js serve`
- DeepAgent 与 strategy 共用同一把 `OPENFINCLAW_API_KEY`（fch_ 前缀），经 Hub Gateway 统一鉴权。

## API Key Safety

- **永远不要把真实 API Key 写进仓库里**（代码、测试、Markdown、提交说明、issue、PR 描述都不行）。示例和文档统一使用占位符：`fch_xxx`、`<fch_...>`。
- **统一一把 Key**：`OPENFINCLAW_API_KEY`（`fch_` 前缀，68 字符）同时驱动 strategy（Hub `/api/v1/skill/*`）与 deepagent（Hub Gateway `/api/v1/agent/*`），两条链路都用 `Authorization: Bearer fch_...` 鉴权。
- **用户层持久化**：`~/.openfinclaw/config.json`（由 `openfinclaw init` 写入；Unix 自动 `chmod 600`）。字段：`apiKey`。
- **MCP 子进程层**：各 agent 平台配置文件里的 `env` 字段（由 `init` 写入；例 `~/Library/Application Support/Claude/claude_desktop_config.json`）。
- 提交代码前检查：`git diff` 不应出现任何像 key 的字符串（`fch_[0-9a-f]{32,}`、`[0-9a-f]{12,}` 等）。
- `doctor` 命令打印 Key 时只显示前 8 位 + `…`，生产行为应当与之保持一致；新代码打印/日志 Key 时务必截断。

## Architecture Principles

- **Core is platform-agnostic**: No OpenClaw, MCP, or agent framework imports in `@openfinclaw/core`. All tool logic is exported as pure functions + JSON schemas.
- **CLI is a thin adapter**: `@openfinclaw/cli` wraps core functions with MCP SDK (Zod schemas) and CLI formatting. Keep it minimal.
- **Config via injection**: Core functions accept `OpenFinClawConfig` interface. Resolution lives in `resolveOpenFinClawConfig()` / `resolveConfigFromEnv()`:
  - Unified key: `--api-key` (CLI) → `OPENFINCLAW_API_KEY` → `~/.openfinclaw/config.json#apiKey`
  - Override config file path with `OPENFINCLAW_CONFIG_PATH`。
  - `resolveOpenFinClawConfig({ allowMissingApiKey: true })` returns a config with `apiKey: ""` instead of throwing — used only by CLI `doctor` so it can render a "key missing" diagnostic. Other commands (含 `deepagent *`) 一律要求 fch_ key，否则 Gateway 返回 401。
- **Unified auth via Hub Gateway**: deepagent 与 strategy 走同一把 fch_ key。strategy 直接打 commons-hub `/api/v1/skill/*`；deepagent 打 Hub Gateway `https://gateway.openfinclaw.ai/api/v1/agent/*`，由 APISIX 完成 fch_ key 校验 + 多租户隔离 + URL rewrite 后转给 DeepAgent 后端。任何 DeepAgent 请求必须走 `deepagentApiRequest()`（默认 base URL 已含 `/api/v1/agent` 前缀），头部用 `Authorization: Bearer ${config.apiKey}`，**不要**再用 `X-API-Key` 或老的 `https://api.openfinclaw.ai/agent` 直连地址。
- **Long-running LLM tools**: 远端 Agent 类长耗时操作采用 submit/poll/finalize 三段式（cross-platform-safe，见 `deepagent/tools.ts` 的 `research_submit` / `research_poll` / `research_finalize`）。**不要**试图在单次 MCP tool call 内阻塞等待 LLM 全部完成 —— MCP 客户端会超时或缓冲。
- **CLI ≠ MCP for streaming**: 终端命令（如 `deepagent research`）可以直接消费 SSE 流做真正 token-by-token 渲染（`process.stdout.write`）；MCP 路径则必须回落到 submit/poll。

## Tool Groups

OpenFinClaw 定位为「以 DeepAgent 为核心的一站式量化交易 agent」—— 行情获取、数据分析、深度报告、策略生成、回测、模拟盘全部由 DeepAgent 承载。本地 FEP 策略开发作为独立的进阶工作流并存。

Tools 分为 2 组，可通过 `--tools=` 独立加载：
- `deepagent` — 14 远端 AI Agent 工具 (fin_deepagent_health / _skills / _threads / _messages / _research_submit / _research_poll / _research_finalize / _status / _cancel / _backtests / _backtest_result / _packages / _package_meta / _download_package)
- `strategy` — 7 本地策略管理工具 (strategy_publish, strategy_validate, strategy_fork, strategy_leaderboard, strategy_get_info, strategy_list_local, strategy_publish_verify)

未传 `--tools=` 时两组全部加载。示例：`--tools=deepagent` 只加载 DeepAgent。想在线先体验 DeepAgent 可访问 <https://hub.openfinclaw.ai/en/chat>。

### DeepAgent 子模块结构（`packages/core/src/deepagent/`）

- `types.ts` — REST 响应 + 7 种 SSE 事件 + `DeepAgentTaskState` 类型
- `client.ts` — `deepagentApiRequest()`（Hub Gateway + `Authorization: Bearer fch_...`）+ `parseDeepAgentSSE()` async generator + 进程内任务态 store (`getDeepAgentTask / listDeepAgentTasks / clearDeepAgentTask`) + `startDeepAgentRun()`（submit 后后台消费 SSE 更新 store）
- `tools.ts` — 14 个 execute 函数 + JSON schemas；研究类走 submit/poll/finalize，其余同步调用

### 设计备注

- **进程级任务 store**：仅保留在 MCP server 进程内存里，进程重启即丢失。DeepAgent 服务端自己持久化 thread/run，CLI 侧不做 SQLite。
- **Paper Engine** 暂未接入。它是独立 API（`api.openfinclaw.ai/engine`）+ 第三套 Key（`OPENFINCLAW_PAPER_API_KEY`，尚未定义）。如要接入，遵循 deepagent 同样模式新建 `packages/core/src/paper/`，并扩展 `OpenFinClawConfig`。

## Adding New Tools

1. Add pure execute function + schema in `packages/core/src/<group>/tools.ts`（`<group>` 可以是 `deepagent` / `strategy` 或新的子目录）
2. Export from `packages/core/src/index.ts`
3. Register MCP tool via `McpServer.registerTool()` (Zod `inputSchema` in the config object) in `packages/cli/src/mcp.ts`，放入对应的 `if (groups.includes("<group>"))` 分支
4. Optionally add CLI command in `packages/cli/src/cli.ts`，复用 `styles.ts` 里的美化 helper

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirror29/openfinclaw-cli](https://github.com/mirror29/openfinclaw-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
