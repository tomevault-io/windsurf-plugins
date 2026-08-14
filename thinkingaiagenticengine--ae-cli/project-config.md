---
trigger: always_on
description: > 本指南由 `CLAUDE.md` 与 `AGENTS.md` 两份**逐字一致**的副本组成：`CLAUDE.md` 供 Claude Code 读取，`AGENTS.md` 供 Codex 及其他 AI agent 读取。**改动任意一份后必须同步另一份**，并运行 `npm run check:agents-docs` 校验一致性。
---

# te-cli 协同开发指南（CLAUDE.md / AGENTS.md）

> 本指南由 `CLAUDE.md` 与 `AGENTS.md` 两份**逐字一致**的副本组成：`CLAUDE.md` 供 Claude Code 读取，`AGENTS.md` 供 Codex 及其他 AI agent 读取。**改动任意一份后必须同步另一份**，并运行 `npm run check:agents-docs` 校验一致性。

## 1. 项目简介 · 谁在用这个 CLI

`@tant/ae-cli`（命令 `ae-cli`）是 ThinkingAI AE 平台的命令行工具，TypeScript / ESM，Node ≥ 18。

**主要使用者既是团队成员，也是 AI agent**（Claude Code、Codex、Cursor 等）。由此引出两条贯穿全文的约束：

- 你写的每条**输出与错误信息都会被 agent 解析**，并据此决定下一步动作。
- 所以输出要结构化、错误要可读可定位（见 §4）。

## 2. 行为准则（先读这一节）

源自 Andrej Karpathy 对 LLM 编码陷阱的观察，在本仓库落地为四条：

1. **先想后写**。新增命令 / flag 前，先确认真实 API path 与请求体——**不要猜**；对照对应 skill 的 reference。出现多种解释时摆出来让人选，不要默默挑一个。
2. **简洁优先**。一个命令就是一个声明式 `Command` 对象 + `execute`。不加没人要的 flag、抽象或兜底逻辑。能 50 行别写 200 行。
3. **外科手术式改动**。照搬现有命令文件的写法，不顺手重构相邻 domain、不改无关格式。只清理你自己改动产生的孤儿代码；发现无关死代码就指出来，别删。
4. **目标驱动**。每次改动先定义可验证的完成标准（见 §7），然后循环到验证通过。

## 3. 构建 · 运行 · 测试

| 命令 | 用途 | 提 PR 前 |
| --- | --- | --- |
| `npm run build` | tsup 打包到 `dist/` | ✅ 必跑 |
| `npm run dev` / `npx tsx src/index.ts` | 本地运行（免构建） | — |
| `npm test` | 冒烟（执行 `--help`） | ✅ 必跑 |
| `npm run verify:*` | 各 domain 工具校验脚本 | 改到对应 domain 时跑 |
| `npm run self-check` | 校验新合入的 CLI 功能合理性 | 建议 |
| `npm run check:release` | 发布门禁（skill frontmatter 等）；CI 发包前必跑 | ✅ 改 skills frontmatter / 发包前 |
| `npm run check:agents-docs` | 校验 CLAUDE.md / AGENTS.md 一致 | ✅ 改本文件时必跑 |

本地起步：`npm install` → `npx tsx src/index.ts --help`。

## 4. 代码与架构范式

### 语言：代码与 CLI 输出统一英文（硬约束）

CLI 同时面向团队成员与 AI agent，**源码内容与所有用户可见输出统一使用英文，不得出现中文或其他自然语言**：

- **代码内容**：字符串字面量（含错误 `message` / `hint`、`desc`、提示文案）与注释，全部英文。
- **CLI 输出**：所有用户可见输出——`--help`、flag `desc`、进度 / 告警（stderr）、错误 envelope 的 `message` / `hint`——全部英文。
- **不翻译 / 不改动**：标识符、JSON 键、命令名、flag 名、字符串插值的变量、URL path、技术词。
- **唯一例外**：真实业务数据本身（如 AE 事件名 `登录` / `支付`）按原样保留——翻译会破坏功能与校验。

不在此约束内：本指南（`CLAUDE.md` / `AGENTS.md`）与提交信息（见 §6）仍用中文。

### 源码布局

| 路径 | 职责 |
| --- | --- |
| `src/core/` | auth、config、client、cli-token、mcp、mcp-access、capability-api（鉴权 / 配置 / HTTP / MCP JSON-RPC / 新 REST API） |
| `src/framework/` | types、register、runner、output（命令框架核心） |
| `src/api/` | 原始 API 访问（`api` 命令） |
| `src/commands/<domain>/` | 各业务域命令（te-analysis、metadata、te-kb…） |
| `skills/` | 给 AI agent 的 skill 包（与命令同步维护） |
| `self-check/` | 自检脚本与发布门禁（`release-gate.mjs` + `checks/`） |
| `tests/`、`test/` | 测试 |
| `scripts/` | 校验 / 工具脚本 |

### Command 对象模式

每个命令是一个 `Command` 对象（定义见 `src/framework/types.ts`）：

```
{ service, command, description, flags[], risk, usesAeHost?, validate?, dryRun?, execute }
```

- `command` 用 `+` 前缀，如 `+query`、`+list_events`。
- 命令文件放在 `src/commands/<domain>/<cmd>.ts`，并在该 domain 的 `index.ts` 里登记到命令数组 + 具名导出。

### CLI / Capability 命名契约

跨 ae-cli 与 common 的命名边界如下：

- ae-cli 命令段和 flag 使用 kebab-case：`metadata data-table property-bindings-update`、`--project-id`。
- capabilityId 使用三段式 dot namespace：`<domain>.<resource>.<action>`。
- capabilityId 每段内部使用 snake_case；common 注册的能力 ID 也遵循这个规则，例如 `metadata.data_table.property_bindings_update`。
- capability gateway 的 `input` / `data` / `meta` JSON 字段使用 snake_case，例如 `project_id`、`data_table_id`、`client_request_id`。
- Java 内部类型、方法和 DTO 字段继续按本地代码风格使用 camelCase；边界层负责转换，不把 camelCase 泄漏到 CLI 公共契约。

映射规则：CLI 层级空格映射为 capabilityId 的点号，CLI 段内 `-` 映射为 capabilityId 段内 `_`。例如 `metadata data-table property-bindings-update` → `metadata.data_table.property_bindings_update`。

### 命令收录与 Gateway 迁移

新增或迁移命令前必须先阅读 [`docs/capability-command-admission.md`](docs/capability-command-admission.md)：

- Gateway 已覆盖的能力默认使用 `capability search/inspect/dry-run/run`，只有具备明确编排价值时进入 L1，具备额外类型、安全或输出价值时进入 L2。
- Gateway 尚未覆盖的必要命令可以暂用现有 transport，但必须按 Transitional 规则记录维护模块、迁移目标、复审日期和退出条件。
- 稳定 ingestion data-plane 是正式 L2 例外：服务端前置网关负责接入安全，CLI 不外发平台凭证，并提供类型化输入、redacted dry-run、日志保护和稳定 transport 测试；满足这些条件的命令不标记 Transitional。
- 现有 `+` 命令不会因新规则被直接删除；Gateway 等价能力上线后再判定保留 L1、迁移 L2 或退回 L3。

### 一切走 RuntimeContext

命令体内**不要裸用 `fetch` / `process.stdout`**，统一通过 `ctx`：

- 取参：`ctx.str(name)` / `ctx.num` / `ctx.bool` / `ctx.json`
- 发请求：`ctx.api(method, path, params, body)`（KB external 接口用 `kbApi`，见下）
- 稳定 ingestion data-plane 使用专用 `RuntimeContext` 方法，不在命令内裸用 `fetch`，也不复用会附加平台凭证或记录正文的通用 client。
- 与 active AE host 无关的稳定 data-plane 命令设置 `usesAeHost: false`，不在子命令帮助中暴露误导性的 `--host` override。
- 上下文：`ctx.host()` / `ctx.token()` / `ctx.service()`
- 输出：`ctx.out(data)`

### flag / risk / dry-run 约定

- 每个 flag **必须带 `desc`**（会被 agent 读），类型 ∈ `string | number | boolean | json`。
- 风险等级标 `risk: 'read' | 'write' | 'high-risk-write'`（与 lark-cli 对齐）——**仅 `high-risk-write` 触发二次确认**，除非用户带 `--yes`；它用于删除/移除，以及取消运行任务、修改权限或认证策略等需要显式确认的高影响写操作。`read`/`write` 不确认。
- 尽量实现 `dryRun(ctx)`，返回 `{ method, url, params, body }`，让 `--dry-run` 能在不真正请求的情况下预览。

### 输出与错误

- 输出 envelope：`{ ok, data, error: { type, code, message, hint } }`，`type ∈ auth | api | validation | config`。
- **JSON 走 stdout，进度 / 告警走 stderr**（用 logger 或 `process.stderr.write`）。

### 鉴权

- **唯一凭证是 CLI token**（`src/core/cli-token.ts` 的 `getCliToken()`），不存在独立的 mcp-token 获取/mint 逻辑。取值优先级：进程内缓存 → `secure-store.cliToken` → 沙箱注入的 `cli-token.json` → 用 accessToken 调 `/v1/ta/cli/token/generate` **mint**（`auth login` 成功后立即 mint 并写回 secure-store；若缺失则在首次命令执行时补 mint）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThinkingAIAgenticEngine/ae-cli](https://github.com/ThinkingAIAgenticEngine/ae-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
