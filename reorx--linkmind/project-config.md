---
trigger: always_on
description: LinkMind 是一个基于 Telegram Bot 的智能链接收藏和分析工具。用户把链接发给 Bot，自动抓取网页内容、生成中文摘要、通过向量搜索发现相关内容，生成 insight。附带 Web 界面浏览。
---

# LinkMind — Project Summary

## What Is This

LinkMind 是一个基于 Telegram Bot 的智能链接收藏和分析工具。用户把链接发给 Bot，自动抓取网页内容、生成中文摘要、通过向量搜索发现相关内容，生成 insight。附带 Web 界面浏览。

## Monorepo Structure

pnpm workspace，三个包：

```
linkmind/
├── core/       @linkmind/core    — 共享类型和工具函数
├── server/     @linkmind/server  — 云端服务（Bot + Pipeline + Web）
└── probe/      @linkmind/probe   — 本地抓取 daemon（SSE 连接云端）
```

- `core/` — `ScrapeData`, `ScrapeRequestEvent`, `ScrapeResultPayload`, `UrlType` 类型定义；`htmlToSimpleMarkdown()`, `isTwitterUrl()` 工具函数。无运行时依赖，通过条件 exports 暴露：`types` 条件指向 `.ts` 源码（IDE/typecheck），`default` 条件指向 `dist/` 编译产物（运行时）。
- `server/` — Telegram Bot、Pipeline（scrape → summarize → embed → related → insight）、Express Web 界面、Probe SSE 事件分发。
- `probe/` — 本地 daemon，通过 SSE 接收抓取任务（Twitter via bird CLI、Web via Playwright + Defuddle），结果 POST 回云端。

## Tech Stack

- **Runtime**: Node.js >= 22, TypeScript (tsx)
- **Package Manager**: pnpm (workspace)
- **Bot**: Telegram Bot API (grammy)
- **Web Scraping**: Playwright + Defuddle
- **LLM**: OpenAI 兼容 API
- **Database**: PostgreSQL (Neon) + Kysely ORM + pgvector
- **Web**: Express + EJS
- **Twitter**: bird CLI
- **Durable Execution**: Absurd SDK

## Architecture

```
                          Cloud (SaaS) — @linkmind/server
┌──────────────────────────────────────────────────────────────┐
│  Telegram Bot ──▶ Pipeline ──▶ PostgreSQL (Neon)             │
│  Web UI (Express+EJS)    │       - links, users, invites     │
│                          │       - probe_events, probe_devices│
│                          ▼       - link_relations            │
│                    SSE Server ◀── Auth: Bearer token          │
└──────────────────────────┬───────────────────────────────────┘
                           │ Events (SSE) ↓  Results (POST) ↑
┌──────────────────────────┴───────────────────────────────────┐
│               Local — @linkmind/probe                        │
│  linkmind-probe daemon                                       │
│    ├── bird CLI (Twitter/X, Chrome cookies)                   │
│    └── Playwright + Defuddle (Web)                            │
└──────────────────────────────────────────────────────────────┘
```

**Pipeline 流程 (process-link)：**

```
Step 1: scrape
  ├─ Probe 数据已有 → 直接使用
  ├─ Twitter URL → 创建 probe_event (url_type: twitter) → waiting_probe → return
  └─ 普通 URL → scrapeStepWithFallback:
      1. Playwright + Defuddle
      2. 字数 < 200? → Playwright 重试
      3. 仍不足? → Firecrawl API (FIRECRAWL_API_KEY)
      4. 仍不足? → Probe fallback (url_type: browser) → waiting_probe → return

Step 2: summarize (LLM)
  → 输出 { valid_content, summary, tags }
  → valid_content: false? → Step 2.5

Step 2.5: re-scrape + re-summarize (仅当 valid_content=false)
  → scrapeStepWithFallback(skipPlaywright=true): 只走 Firecrawl → Probe
  → 重新 summarize

Step 3: embed (向量化 summary)
Step 4: related (向量搜索相关 records)
Step 5: insight (LLM，基于 summary + related links)
```

Probe 等待机制：record 进入 `waiting_probe` 状态，probe 端通过 SSE 接收任务，抓取后 POST 回结果，触发 `handleProbeResult()` 恢复 pipeline。

## Common Commands

```bash
# 安装依赖
pnpm install

# 构建（core + server）
pnpm build

# 类型检查（server + probe）
pnpm typecheck

# 运行测试
pnpm test

# 启动 server（开发，tsx）
pnpm --filter @linkmind/server run dev

# 启动 server（生产，需先 build）
cd server && node dist/index.js

# 启动 probe（开发）
pnpm --filter @linkmind/probe run dev -- run --foreground

# CLI 脚本（开发，tsx 直接运行）
cd server && npx tsx src/cli.ts <command> [args]

# CLI 脚本（编译后）
cd server && node dist/cli.js <command> [args]

# 列出所有可用 CLI 命令
cd server && node dist/cli.js
```

## 部署

- 部署配置**不在本仓库**，位于 OpenClaw workspace 的 `deploy/` 目录下
- 使用 **Ansible** 管理所有部署操作，playbook 和 roles 都在 `deploy/ansible/`
- 服务器：hh-hk-01 (103.69.129.33:1122)
- 所有与部署相关的改动都在 workspace 的 `deploy/` 目录进行，不要在本仓库创建部署文件

### Deployment — launchd (本地开发)

Server 通过 macOS launchd 作为 user agent 运行。

**plist 路径**: `~/Library/LaunchAgents/com.linkmind.plist`

```bash
# 加载 / 卸载
launchctl load ~/Library/LaunchAgents/com.linkmind.plist
launchctl unload ~/Library/LaunchAgents/com.linkmind.plist

# 启动 / 停止
launchctl start com.linkmind
launchctl stop com.linkmind

# 查看日志
tail -f ~/Code/linkmind/data/launchd-stdout.log
tail -f ~/Code/linkmind/data/launchd-stderr.log
```

注意：`KeepAlive=true`，`launchctl stop` 后会自动重启，彻底停止需 `unload`。

## 生产数据维护

- **禁止对生产环境执行裸 SQL 操作**
- 所有数据维护必须通过 `server/src/cli/admin-*.ts` 脚本完成
- 流程：
  1. 在 `server/src/cli/` 下编写 TypeScript 脚本，调用项目内部函数
  2. 先用本地 `.env` 测试
  3. 确认无误后，使用 `.env.prod` 对生产环境执行：
     ```bash
     cd server
     # 开发环境（tsx 直接运行）
     npx tsx --env-file=.env.prod src/cli.ts <command> <args>
     # 或编译后
     node --env-file=.env.prod dist/cli.js <command> <args>
     ```
- `.env.prod` 包含生产环境配置，已在 `.gitignore` 中，不会提交到仓库

## 生产 Migration 安全验证流程

在生产环境执行 migration 前，先在本地复制生产数据库进行验证。

### 步骤

```bash
# 1. 复制生产数据库到本地（需要本地 PostgreSQL 运行中）
#    脚本读取 server/.env.prod 中的 DATABASE_URL，dump 后 restore 到本地
#    注意：需要 superuser (reorx) 创建数据库和安装 pgvector 扩展
export PATH="/opt/homebrew/opt/postgresql@18/bin:$PATH"
LOCAL_DB="linkmind_pro_$(date +%Y%m%d)"

# dump 生产数据
pg_dump "$(grep '^DATABASE_URL=' server/.env.prod | sed 's/^DATABASE_URL=//')" \
  --format=custom --no-owner --no-privileges -f /tmp/${LOCAL_DB}.dump

# 创建本地数据库（superuser）+ 安装 pgvector

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reorx/linkmind](https://github.com/reorx/linkmind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
