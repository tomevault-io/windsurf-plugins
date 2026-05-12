---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 架构一句话

三个独立进程（`ingest live` / `dispatcher` / `ingest backfill`）共享一份 WAL 模式的 SQLite (`data/wechat-oracle.db`)。**WeFlow 的 HTTP API 是上游唯一真相源**——live 走 SSE 推流，backfill 导 WeFlow JSON 导出，没有路径直连微信原始 DB。dispatcher 轮询 DB，命中 `@<bot>` 文本后过 OpenAI-compatible LLM，再用 wx4py（Windows UI 自动化）把回复打回群。

数据形态：`messages`（主表）+ `forwarded_records`（合并转发子项，`parent_msg_id` 反指）+ `command_runs`（dispatcher 幂等记录，`msg_id` 主键）。所有写入路径必须走 `ingest/writer.py:write_messages`，靠 `UNIQUE(dedupe_key)` 跨源去重。详细字段语义看 `schema.sql`（DDL 行级注释是主源）+ `models.py` docstring。

## 常用命令

依赖管理 / 装环境（项目用 `uv`，Python ≥ 3.12）：

```bash
uv sync                                # 装依赖
uv run wechat-oracle init-db           # 建表（幂等，每次跑前先来一遍也无所谓）
uv run wechat-oracle status            # 查总条数 / 按 status / 按群分布——快速 health check
```

三个长跑 / 一次性进程（生产路径上 `live` + `dispatcher` 同时常驻）：

```bash
uv run wechat-oracle ingest live                                 # SSE 实时抓 → DB
uv run wechat-oracle dispatcher                                  # DB 轮询 → LLM → wx4py 回群
uv run wechat-oracle ingest backfill <path.json> --format weflow # 一次性导入 WeFlow JSON 导出
```

诊断 `WO_GROUPS` 解析（群名找不到对应 wxid 时用）：

```bash
uv run wechat-oracle weflow find <群名 / 备注 / wxid 片段>   # 同时查 contacts + sessions
uv run wechat-oracle weflow sessions --groups-only          # 列出所有 @chatroom 会话
```

封装好的 wrapper（POSIX）：`scripts/import.sh <export.json>` / `scripts/track.sh`；Windows 对应 `.bat` 同名文件。

**测试 / lint：本项目当前没有 pytest、ruff、mypy 或任何 CI 配置**——`pyproject.toml` 里没有 dev-dependencies 段，仓库里也没有 `tests/`。改代码要自验，靠跑上面那几条 + 看 `data/dispatcher.log` / `data/llm_debug.log`。要加单测的话先和用户对一下要不要顺便引一套 pytest。

## 平台前提

**生产环境是 Windows + 中文 WeChat 4.1.x（Qt 版）+ WeFlow 桌面端**——wx4py 走 UI 自动化只在 Windows 工作，dispatcher 的「发回群」分支在 macOS/Linux 上跑不起来。但**导入 / 查询 / DB 操作（init-db / backfill / status / weflow find）跨平台**，本仓库的 dev worktree 多半就在 macOS 上。改 dispatcher 时如果 Windows 不在手边，至少要确保 `parse_command` / SQL 构造 / LLM 调用这些纯逻辑路径能在本地手动 invoke 验证。

## 文档导航（新会话先读这段）

- **本文件**：项目特定的契约 / 用户偏好 / 踩过的坑（Lessons Learned）。冲突时**项目级 > 全局基线**。
- **`README.md`**：对外用户文档。改命令 / 配置 / schema / 入口都要回头同步。「数据流细节」段对理解三个进程怎么协作最有帮助。
- **代码内 docstring**：模块顶端有 5–10 行职责说明，`dispatcher.py` / `live.py` / `forwarded.py` 写得最详细。
- **`schema.sql`**：DDL 行级注释是字段语义的**主源**，发现描述模糊就在这儿改。
- **`.Codex/hooks/check_doc_sync.py`**：自动 backstop，跨文件漂移会反向提醒。

按场景找入口：

| 我要改… | 先读 | 同步目标（见「易漂移点速查」） |
|---|---|---|
| slash 命令的语法 / 加新命令 | `dispatcher.py` 顶端 docstring + 「命令体系维护契约」 | F4 |
| 入库字段 / 新加列 | `schema.sql` + `models.py` + 「管道字段对齐」 | F1 / F2 |
| 配置项 | `config.py` `Settings` | F3 |
| CLI 子命令 | `cli.py` | F5 |
| 新 importer / 数据源 | `ingest/writer.py` 的 `write_messages` 入口 | F1 / F11 / 管道字段对齐 |
| WeFlow API 字段语义 | `live.py` 顶端 + `forwarded.py` 顶端 docstring | F11 / Lessons Learned |
| dispatcher 查询逻辑 | `dispatcher.py` 的 `fetch_candidates` | F2（要 UNION forwarded_records） |

## 用户偏好

- **先想后动**：动手前先把意图、权衡、影响范围想清楚再写代码。
- **简单但不简陋**：选最直接的实现，能用一个函数说清楚的事不要拆三层。
- **不堆错误处理 / 冗余函数**：除非确有必要或用户明说，不要预防性地加 try/except、参数校验、辅助包装。健壮性靠数据契约和显式失败，不靠到处兜底。
- **改动有取舍时先问用户**：涉及策略选择（路径布局、复制 vs 引用、是否删数据等），列出 trade-offs 让用户拍板，不擅自决定。
- **destructive 操作必须确认**：删 DB 行、清目录、`git reset --hard` 之类，先讲清后果再执行。

## 项目特性约定

- **数据本地优先**：`data/` 是项目自有归档，导入时把媒体复制进来（`data/media/<group_id>/<kind>/`），不留对外部路径的依赖。
- **跨源去重**：所有写入路径走 `write_messages()` → `UNIQUE(dedupe_key)`，新增 importer 时复用、不要绕过。
- **`source` 字段记录管道来源**（`live` / `backfill`），不要用它表达消息状态——状态走 `status` 列。
- **WeFlow 是唯一真相源**：实时抓和历史回灌都过 WeFlow，不直接读微信原始 DB。
- **dispatcher 冷启动不回放历史**：`_skip_backlog` 在启动时把所有未处理的 `@bot` 历史消息标 `(startup-skip)`，避免冷启动 / 大批量回灌后向群灌一通陈年答复。改这个行为要同步 README 数据流段。

## 易漂移点速查

同一个事实被多处编码就会漂移。下表列出本仓库所有冗余存储位置；改任何一处，**必须**主动看其它行的状态。`.Codex/hooks/check_doc_sync.py` 覆盖 F1–F5，其它行靠人 + 本契约保障。

| ID | 事实 | 存储位置 | 自动校验 | 注意点 |
|---|---|---|---|---|
| **F1** | `messages` 表 schema | `schema.sql` + `models.py:Message` + `ingest/writer.py:INSERT_SQL/_row` + `README.md` 数据库段 | hook | 加列要四处全改 |
| **F2** | `forwarded_records` schema | `schema.sql` + `models.py:ForwardedItem` + `ingest/writer.py:INSERT_FWD_SQL` + `ingest/forwarded.py` 解析 + `README.md` | hook | 加 datatype 占位时同步 `_PLACEHOLDER` |
| **F3** | `WO_*` 配置项 | `config.py:Settings` + `README.md` 配置参考表 + `README.md` 快速上手 .env 示例 | hook | 删字段记得 .env 例子也清掉 |
| **F4** | slash 命令集 | `dispatcher.py` `Command` 子类 + `README.md` 命令详解 | hook | `Command.help()` 自动从类 attr 生成；改 attr 时同步 README |
| **F5** | CLI 子命令 | `cli.py` typer 命令 + `README.md` 快速上手 + 三进程表 | hook | 加进程要同步 README 三进程表 |
| F6 | localType→MsgType 映射 | `ingest/backfill.py:_WEFLOW_LOCAL_TYPE_MAP` + `ingest/forwarded.py` 常量 | — | live.py 通过 import 复用，单源 |
| F7 | `MsgType` enum 值 | `models.py:MsgType` + `schema.sql` `type` 列注释 + `dispatcher.py:fetch_candidates` SQL CASE 占位映射（image/voice/video/sticker） + `README.md` schema 段 | — | 加新 type 要同步：enum、CHECK、SQL CASE 占位（如新 type 经常无 content_text）、README。**fetch_candidates 不再按 type 过滤**——所有类型都进候选池 |
| F8 | 状态枚举（`Status` / `source` / `command_runs.status`） | `models.py` enum + `schema.sql` CHECK + 字面量散布 + `README.md` | SQLite CHECK 兜底 | CHECK 失败时是 IntegrityError，不会静默 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NotDesigned/WeChat-Oracle](https://github.com/NotDesigned/WeChat-Oracle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
