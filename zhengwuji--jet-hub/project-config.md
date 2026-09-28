---
trigger: always_on
description: - **推理输出**（thinking / reasoning）一律使用中文。
---

# 项目指令：dsh-codearts-auth

## 语言约束

- **推理输出**（thinking / reasoning）一律使用中文。
- **正文输出**（正文回复、代码注释说明、总结、文档）一律使用中文。
- 代码标识符、关键字、类型名称、变量名等保持英文不变。

---

## 📚 分册索引（必读）

本文件只保留**规则与约定**。完整证据、推导过程、实测数据、排查脚本与历史缺陷
记录在下列分册中 —— **改动相关代码前请先读对应分册**。

> ⚠️ **规则本身在本文件里是完整的**。分册补充的是「为什么这么定」与「出错时怎么查」。
> 出现异常现象、或要推翻某条规则时，**必须**先读分册，别重复已记录过的排查路径。

| 分册 | 覆盖内容 | 何时必读 |
|---|---|---|
| [`docs/agents/trae.md`](docs/agents/trae.md) | TRAE 四条协议的坑、通道路由、推理档位、Max 模式、图片判定 | 改 `src/trae*.ts` |
| [`docs/agents/pricing.md`](docs/agents/pricing.md) | 计费倍率解析差异、`maxOutputTokens` 下发、同名消歧、X-Domain | 改展示名 / 请求头 / 输出上限 |
| [`docs/agents/credits.md`](docs/agents/credits.md) | 五个 provider 的签到协议、幂等判据、能力矩阵门控 | 改 `src/*-credits.ts` / `credits-capabilities.js` |
| [`docs/agents/antigravity.md`](docs/agents/antigravity.md) | 两条通道选路、八条防封号硬性约束、协议字段位置 | 改 `src/antigravity*.ts` |
| [`docs/agents/catalog-gating.md`](docs/agents/catalog-gating.md) | 模型黑名单、账号门控、`listAllModels` 契约、两步式登录 | 改 `listModels` / `model.list` RPC |
| [`docs/agents/qoder.md`](docs/agents/qoder.md) | Qoder 积分端点实测、幂等判据、`openai-compat.ts` 边界 | 改 `src/qoder*.ts` / `openai-compat.ts` |

---

## 项目概述

本项目是 DeepSeek Harness 的插件（`dsh-codearts-auth`），提供华为云 CodeArts
浏览器登录与凭据管理，并作为多服务商统一接入网关。

**11 个 provider**，分属 5 套互不相同的协议族：

| 协议族 | provider | 特点 |
|---|---|---|
| 腾讯 CodeBuddy 系 | `buddy` / `buddy-intl` / `workbuddy-cn` / `workbuddy` | 同一 CLI 内核与认证协议，差异**全在 `endpoint`** |
| 有道 LobsterAI | `lobsterai` | 本地回调 + authCode 换 token |
| 阿里 Qoder | `qoder` / `qoder-cn` | PKCE 设备码轮询 + **加密推理端点**（WASM 签名） |
| 字节 TRAE | `trae` / `trae-intl` | ExchangeToken 轮换 + 载荷双向转换（OpenAI ↔ SOLO） |
| 华为 CodeArts | `codearts` | `SDK-HMAC-SHA256` 签名 |
| Google Antigravity | `antigravity` | 本机凭据复用，**不进账号池** |

⚠️ **区域版各占一个 provider**：CodeBuddy `buddy`(国内)/`buddy-intl`(国际)、
WorkBuddy `workbuddy-cn`(国内)/`workbuddy`(国际)、Qoder `qoder`(国际)/`qoder-cn`(国内)、
TRAE `trae`(国内)/`trae-intl`(国际)。两侧端点与登录态**互不相通**，凭据各自独立。

### 协议族的硬性差异（改代码前必读）

- **CodeBuddy 系**：`endpoint` 中国版 `copilot.tencent.com` / 国际版
  `www.workbuddy.ai`、`www.codebuddy.ai`。**模型池由 endpoint 决定**，
  故不可当作全局常量。相邻产品共用一个适配器类，差异由产品配置承载。
- **LobsterAI**：与腾讯系**完全不同源**（登录方式、请求头、续期载荷、签到流程、
  版本号来源都不同）。**不共用 `BuddyProduct` 类型** —— 其中 `apiDomain` /
  `productCode` / `attributionName` / `userAgentByModelFamily` /
  `appendSessionParams` 对它全部无意义。独立一套 `src/lobsterai*.ts`。
- **Qoder**：独立一套 `src/qoder*.ts`。**两条推理路径认两套模型名且 host 不同**
  （加密走 `api2.qoder.sh` 认目录 key / 公开走 `api2-v2.qoder.sh` 认通用名，
  混用 404）。加密请求体的签名头**必须原样透传**，用 `Bearer` 覆盖会被判签名无效。
  ⚠️ 改 `qoder-wasm.ts` 前先读**不入库**的 `docs/qoder-encryption-notes.md`。
- **TRAE**：独立一套 `src/trae*.ts`，**请求体与响应都要转换**。详见
  [trae 分册](docs/agents/trae.md)。
- **`src/openai-compat.ts` 只服务 qoder**。`buddy-adapter.ts` /
  `lobsterai-adapter.ts` **刻意不改用它** —— 那两份已被大量单测与线上流量验证，
  重构属无关高风险改动。

- **包名**：`dsh-codearts-auth`
- **入口**：`lib/index.js`（宿主侧）、`lib/client/jet-hub.js`（客户端 bundle）
- **构建**：`pnpm build:all`（`tsc` + `esbuild` + `.wasm` 资源复制）
- **语言**：TypeScript　**许可**：MIT

## 技术栈与约束

- **Node.js**：`^22.19.0 || >=24.0.0`
- **构建系统**：宿主侧 `tsc` → `lib/`；客户端 `esbuild`
  （`plugin-src/client/build.mjs`）→ `lib/client/jet-hub.js`；
  `scripts/copy-assets.mjs` 复制 `.wasm`（**`tsc` 不搬非 TS 资源**）。
  三者都产出到已 gitignore 的 `lib/`，`prepare` 执行 `pnpm build:all`。
- **测试**：Vitest
  - `pnpm test` — 单元测试（快速，无网络，全部 mock）
  - `pnpm test:e2e:*` — 端到端，按 provider 分列，**均有闸门默认跳过**，
    哪些消耗模型积分见 `tests/e2e/README.md`
- **依赖管理**：pnpm workspace（作为 DSH 插件安装）
- **代码风格**：与 `@deepseek-ai/dsh` 主仓库保持一致

## 项目结构

| 路径 | 说明 |
|-------|------|
| `src/` | TypeScript 源码目录（宿主侧） |
| `plugin-src/client/` | Jet Hub 客户端源码（esbuild 打包） |
| `lib/` | 编译产物（已 gitignore） |
| `tests/unit/` | 单元测试 |
| `docs/agents/` | **AGENTS.md 分册**（见上方索引） |
| `cordis.patch.yml` | DSH bundle 补丁 |
| `scripts/` | 构建辅助、只读探针脚本 |

## DSH 插件契约

- 插件使用 `@deepseek-ai/dsh` 的 `credentials`、`commands`、`llm` 服务注入
- 凭据存储使用 `ctx.credentials`，ref 格式遵循 POSIX 标识符
- LLM provider 通过 `ctx.llm.registerAdapter()` / `registerConfigurableProviders()` 注册
- 插件配置通过 `ctx.schema` 在 profile layer 栈中声明

## 工作方式

所有 `ctx.xxxAuth` 服务遵循统一接口：

- `login(options?)` — 执行浏览器登录流程
- `startLogin(options?)` — 两步式登录（先返回 loginUrl，Jet Hub 据此弹窗）
- `refreshAccountCredential(refName)` — 按凭据 ref 续期**指定账号**
- `refreshAll(pool)` — 批量续期全部账号（定时调度器）

⚠️ **不注册任何斜杠命令**：所有 provider 的登录/状态/续期**全部**在 Jet Hub 完成。

⚠️ **CodeArts 只支持账号池，单凭据模式已移除**：

- 凭据一律存 `CODEARTS_ACCOUNT_XXX`；固定的 `CODEARTS_ACCESS_TOKEN`
  **不再被写入或读取**（常量保留仅为兼容 `refName` 缺省值）。
- 只服务于单凭据路径的方法**已删除**：`status()` / `refresh()` / `logout()` /
  `scheduleRefresh()` / `scheduleModelRefresh()`。`refreshModels()` **签名改为接收 `pool`**。
- `codearts-login` / `codearts-status` / `codearts-refresh` 三个命令**已删除**
  （代码里**从来没有** `codearts-logout` 命令，logout 只是服务方法）。
- 门控判据因此**完全一致**：都只看账号池，`extraCredentialRefs` 参数已删除。
- 老用户影响：若此前只用固定 ref 登录过，模型列表会变空，需在 Jet Hub 重新登录一次。

### ⚠️ 续期不得按 `enabled` 过滤

`refreshAll()` 与 `src/index.ts` 的续期调度器**只按 `refreshable` 过滤，不看 `enabled`**。

停用只应影响「账号池的自动选号」，与「凭据是否需要保持新鲜」无关。

**真实缺陷**：两处都按 `enabled` 过滤 →
`refreshAll()` 里停用期间 refresh_token 一路放到失效；
调度器的 `accounts.some(a => a.refreshable && a.enabled)` 让
**所有账号都停用时续期定时器根本不启动**。用户重新启用后拿到死凭据，只能重新登录。

所有 provider 的 `refreshAll` 与调度器**都必须保持只看 `refreshable`**。

### ⚠️ 客户端列出的 provider 必须与服务端注册**一一对应**（面板不得成空壳）

**铁律**：`plugin-src/client/jet-hub.js` 的 `PROVIDERS` 列表里**每一个** id，

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhengwuji/Jet-Hub](https://github.com/zhengwuji/Jet-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
