---
trigger: always_on
description: > 本文件帮助新一轮 AI 会话快速理解 StoryForge 仓库。
---

# CLAUDE.md — StoryForge 项目上下文

> 本文件帮助新一轮 AI 会话快速理解 StoryForge 仓库。
> 上位规范见 `docs/internal/AGENTS.md`、日常执行版见 `docs/internal/AI_ITERATION_GUIDE.md`。
> 当前阶段事实以 `docs/internal/current-phase.md` 为准；下一步入口见 `docs/internal/TODO.md`。

## 1. 项目定位

StoryForge 是面向**长篇小说生产**的可验证创作流水线：
每一次生成、检索、评审、修复、批准与回写，都必须留下可追溯证据，而不是只产出一段孤立文本。

设计立场：**先做诊断控制台，再做生成器**。任何生成路径都先有读取证据 → 评审 → 修复 → 批准的闭环，再考虑接真实模型。

## 1.1 当前项目真相（2026-07-11）

- StoryForge 当前处于**Desktop 对话式 Agent 与私测 Alpha 收口阶段**。
- 产品定位（2026-06-24 拍板）：**作者辅助 IDE**，不是自动长篇生产器；`apps/desktop` 是唯一主产品体验；`apps/web` 已退场（2026-06-21 完成收口），不再作为维护、调试、兼容或契约验证入口。
- 交互中枢（2026-06-30 拍板）：**对话式 agent**；批量自动整书不再是主线，BookRun 降级为 managed Writing Run 的内部兼容实现与后台工具。
- 2026-07-01 已合并：单色调明暗双主题 UI 改版（PR #42）；私测 Alpha 单机后端——PyInstaller sidecar exe 独立起服、BYO-key、`llm-provider.json` 写盘换模型即生效、NSIS 安装包内嵌 sidecar（PR #43/#44）；中间交互区收口为对话式 Agent，`chat.explain` 接真·LLM，对话从文件级解绑为项目级（PR #46）。
- 2026-07-02 已合并：左栏会话历史列表接真后端 + 欢迎页输入框接真发送（PR #48）；Agent loop 三步落地——path-scoped 只读 `fs.list` / `fs.read` / `fs.search`（PR #49）、chat 自由文本走 LLM 工具循环（最多 8 轮、失败回落单轮，PR #50）、前端流程树全事件驱动删预制骨架步骤（PR #51）。
- Agent loop 边界：工具循环入口是 chat 自由文本；审稿 / 修订 / 新文件起草 / 一致性观察 / 深度一致性已作为循环内工具并入（`file.review` / `file.revise` / `file.create` / `project.consistency` / `project.deep_consistency`，一次对话最多一个 proposed patch，机械观察工具不下结论、语义评审工具只出 advisory 信号），显式按钮路径仍走固定管线；chapter.review / bookrun.* 绑定 DB 实体、BookRun 定位后台工具，不并入循环（已记为决定）；语义 judge 已从 `os.getenv` 迁 `resolved_llm_env`（下沉 `app/common/llm_env.py`，吃 `llm-provider.json` 覆盖链）；真·LLM tool-calling headless 实跑已通过（2026-07-02，deepseek-v4-flash，证据 `.codex/real-llm-agent-loop-*` 五个目录，深度一致性 6 处埋雷全中），真机 GUI 渲染观感未验；写回红线见下条（2026-07-31 已随权限档位改写）。
- 2026-07-04 已合并（蓝图 W1「live 循环语义收口」，PR #70，schema 冻结下零 ORM 变更）：**F09** live 工具循环每轮开头读 `run.status`，作者点暂停 / 停止即收尾不再烧新一轮 BYO-key（不 append / 不 complete，status 保持控制通道写入的 stopped/paused），起服收尸非终态 run（`reap_non_terminal_agent_runs`，failed + reason=process_restart，**仅 sqlite 单进程 sidecar 收尸**）；**F10** 完成 / 失败事件 payload 富化 + 终态流事件，前端超时改「close socket → 后台轮询事件表重建终态」（不再硬 reject，纯函数 `reconstructAgentResultFromEvents`）；**F11** `intent._detect_intent` 中文关键词表下线，固定管线只认显式 intent + 结构化参数，自由文本一律落 chat.explain 循环；**sidecar 版本握手**（taskkill+respawn）：`/health/ready` 暴露 `app_version`，Tauri 起服比对版本不符即强杀旧孤儿 sidecar 重启。真机 GUI 多轮渲染 / 点停止桌面观感 / 超时转轮询实取回 / 版本握手实机验证均归 E2E-1 真机清单未验。
- 2026-07-04 已合并（蓝图 W2「sqlite schema 单一事实源」，唯一定时炸弹 F01 拆除）：sidecar 起服由 `bootstrap_sqlite_database` 跑 alembic 收口——已纳管库 `upgrade head`、存量 create_all 库（无 alembic_version）走「SQLite backup API 备份（`*.pre-alembic-<版本>.bak`，保留 3 份）+ `PRAGMA quick_check` 失败即中止 + create_all 补表 + 补 agent_run_events 唯一索引 + `stamp head`」纳管、全新库 create_all + `stamp head`；`alembic/env.py` 支持注入连接 + SQLite `render_as_batch`；alembic 脚本经 `--add-data` 打进冻结 exe（`app/db/migrations.py` 兼顾源码 / `_MEIPASS` 定位）；`create_all` 保留为 SQLite 建表器与收口失败回退。**F01 定时炸弹拆除实证**：已纳管库缺列时起服 `upgrade head` 把列补回（fixture `test_managed_db_applies_pending_migration` 绿）。此后 ORM 列变更走 batch 安全迁移，schema 冻结解除（见 §6 新规矩）。真机「旧版 NSIS 存量库换新 exe 起服 + 会话史完整」归 E2E-1 未验。
- 2026-07-04 已合并（蓝图 W3 首刀「LLM 单一 chat 通道」，拆 high 级 F16 核心）：chat/completions 出网收敛到唯一模块 `app/common/llm_client.py`（自 book_runs 原样下沉带重试 urllib 客户端 + 双鉴权 + 记账；errors 改由该模块定义 `LLMError`/`LLMConfigError`，`book_runs/errors.py` 别名同一类对象、`except`/`isinstance`/502·422 零改动）；**F16 靶心**——`agent_runs/loop_runtime.py` live 循环改吃 common 通道、不再 import book_runs；**真 bug 修复**——`story_state/semantic.py` grounding 配置从裸 `os.getenv` 改吃 `resolved_llm_env` 覆盖链（此前漏迁，sidecar 下读不到 `llm-provider.json` → grounding 静默失活）；密钥脱敏 `redact_secrets` 落 judge/story_state 失败日志；ruff `TID` banned-api 禁裸 `urllib.request` 另起碎片化 chat 客户端。本刀不动：judge/story_state 仍走 httpx（只统一配置源 + 脱敏，未统一传输）、retrieval embedding/reranker、workflow 第 7 客户端（W5 将删）、usage 记账 + 三客户端一致性矩阵（F16 后续）。真 key headless 复跑归真跑轨未验。
- 2026-07-04 已合并（蓝图 W4 batch-1「死域冻结隔离」，拆 F04）：新增 `apps/api/app/domains/DOMAINS.md` 三档清单（live/backing/frozen，**新会话第一入口**，§5 指路）；discovery-first 逐域实证后只卸载 4 个零耦合 frozen router（`analytics`/`batch_refinery`/`collaboration`/`commercial`），护栏 `test_api_surface.py::test_frozen_domain_routers_stay_unmounted` 可证伪、回滚=加回一行 `include_router`；契约 paths 109→100（zero added）、e2e 21/21、全量 847 passed。**冻结只卸 router 不删 `models.py`**（9 域是 models-only/service-live，目录必留）；物理删除 + batch-2 留后续。
- 2026-07-04 已合并（蓝图 W5 core「workflow 分层 prompt 迁入 API」，修 F05 装机死路，schema 冻结下零 ORM 变更）：workflow 的**纯函数**分层 prompt 构建器（7 文件）+ 技能审计投影（`skills/audit.py`）迁入进程内包 `app.domains.book_runs.prompts/`，拆掉两座 importlib 文件路径桥（`workflow_prompt_bridge` / `workflow_skill_audit_bridge`，`git rm`），随 `collect_submodules('app')` 打进冻结 exe。旧桥指相邻 `apps/workflow` 目录、装机 exe 内不存在会在 bookrun.start 才炸；现 `book_generation` 起服链模块级依赖新包、漏打即起服炸。`main.py` 加起服自检 `prompt_layer_bundled`，daily/packaged 两档 sidecar-smoke 断言（**packaged 冻结 exe 实测绿：`分层 prompt 构建器已随 exe 打包(F05 死路已收口)`**）。全量 847 passed（= W4 基线零回归）、ruff 绿、e2e 21/21。**本刀不做**：`apps/workflow` app 物理删除 + 第 7 LLM 客户端删除（W5 高风险步，留后续；prompts 暂在 api/workflow 双存，api 是 live 唯一装机路径）。真机「装机 exe → bookrun.start 真装配」归 E2E-1。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XZZKANY/StoryForge](https://github.com/XZZKANY/StoryForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
