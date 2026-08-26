---
trigger: always_on
description: 本文件给后续在本仓库工作的 AI agent 使用。先读 `README.md`，再按本文约定修改代码。
---

# AGENTS.md

本文件给后续在本仓库工作的 AI agent 使用。先读 `README.md`，再按本文约定修改代码。

## 项目概览

VulnHunter 是一个白盒审计 Agent 平台：导入 GitHub 仓库或源码 zip 后，按 Recon、启发式 Worker（历史漏洞收集完毕后）和/或快速扫描和/或历史漏洞绕过、Reviewer 流程进行漏洞挖掘与验证；Reviewer 默认仅静态复核，可三选一关闭 / 靶场动态 / 局部验证；可选开启 Verifier，在 Reviewer 确认前台漏洞后用 FOFA 搜索同款目标并复测；可选开启攻击链串联，在挖掘与审核结束后根据已确认漏洞尝试多步利用（有本地 Docker 靶场时对无用户交互的链动态验证并落盘脚本）。

- 后端：`backend/app`，FastAPI + SQLAlchemy + SQLite，负责项目导入、阶段调度、Agent 循环、工具注册、报告与漏洞数据。
- 前端：`frontend`，Vite + React + TypeScript + Tailwind，用于审计项目、实时日志、阶段报告、漏洞列表和设置页。
- 模板：`templates`，阶段报告和提示词相关模板。漏洞中文报告对齐 `templates/vuln-report.md`，英文 GitHub Advisory 填表稿对齐 `templates/vuln-advisory.md`（写入 `vulns/{id}/advisory.md`），CVE 5.2 JSON 对齐 `templates/cve.json`（写入 `vulns/{id}/cve.json`；Agent 用 `ReadCveRecord` / `SetCveRecordField` 逐字段填写，未知字段统一占位符 `VULNHUNTER_PENDING`）。
- 运行态数据：`data/projects/{id}`、`data/logs`、`data/app.db`。除非任务明确要求，不要手工改运行态数据或提交生成文件。例外：仓库自带展示案例 `data/projects/11`（MemoBoard / `vulnhunter-python-lab`）及其 `showcase/db-seed.json`；`init_db()` 会幂等导入对应 DB 行，可用 `VULNHUNTER_DEMO_SEED=0` 关闭。不要提交整份 `data/app.db`。

## 常用命令

Windows：

```bat
start.cmd
stop.cmd
scripts\run-tests.cmd
```

Linux / macOS：

```sh
sh start.sh
sh stop.sh
sh scripts/run-tests.sh
```

手动后端（Windows 用 `.venv\Scripts\activate`，Unix 用 `. .venv/bin/activate`）：

```bat
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --reload-dir app --timeout-graceful-shutdown 2 --host 127.0.0.1 --port 16780
```

手动前端：

```bat
cd frontend
npm install
npm run dev
npm run build
```

后端测试：

```bat
cd backend
.venv\Scripts\activate
pytest
```

## 后端约定

- API 路由放在 `backend/app/api`，Pydantic schema 放在 `backend/app/schemas.py`，数据库模型放在 `backend/app/models.py`。
- 使用 `SessionLocal` 时保持短生命周期，优先用 `with SessionLocal() as db:`；测试会 monkeypatch 已导入的 `SessionLocal`。
- SQLite schema 变更要更新 `models.py` 中的模型，并在 `_ensure_columns()` 里补充已有库的兼容列迁移。
- 项目路径相关逻辑优先使用 `backend/app/services/paths.py`，不要散落拼接 `data/projects/{id}`。
- 阶段调度、暂停/恢复/取消逻辑集中在 `backend/app/services/pipeline.py`；Agent 循环相关逻辑在 `backend/app/agent`。
- 可重置启发式 Worker 挖掘进度：`POST /api/projects/{id}/reset-progress`，仅暂停或终态（completed/cancelled/error）可用。清文件 `audited`/认领/启发式轮次摘要与 Worker 检查点；快速扫描 Sink 队列与历史漏洞绕过进度不重置，Semgrep 产物与冻结名单保留。保留漏洞产出、侦察文档、定权/跳过和环境；重置后保持暂停，便于换模型或改挖掘模式后再续跑。
- 统一接续对话：项目详情「阶段日志」SSE 面板下方提供输入框，按当前小阶段 Tab（地图/鉴权、启发式、环境搭建等）操作。**接续**=用该小阶段最新一轮完整消息继续；**新开**=放弃可恢复检查点并按阶段规则再跑一轮（可附用户说明）；该轮**进行中**时输入为**引导**，在下一轮模型调用前注入（不中断正在执行的工具/LLM 请求）。每小阶段结束后检查点归档至 `workspace/last-conversation/`。API：`GET/POST /api/projects/{id}/conversation`（`log_phase`、`action=steer|continue|new`、`message`）。旧 `recon-subphases/.../rerun` 与 `lab-setup/retry` 仍可用，前端已收口到 Composer。地图新开更新须 Agent 调用 `FinishReconMap` 结束。项目级「全部暂停 / 全部续跑」保留；大阶段级暂停/续跑/新跑已移除。
- 项目挖掘模式（赏金模式 `bounty` / 全量模式 `full` / 自定义模式 `custom`）在创建时确定，默认赏金模式；创建后仅当项目暂停或完成才可更改。已完成项目保持 `completed`，不可点暂停改成 `paused`。赏金/全量规则与闸门在 `backend/app/audit_mode.py` 和 `backend/app/prompts/modes/`；自定义模式在设置页维护命名提示词库，项目选用时快照正文，无赏金代码硬闸门，完全依赖提示词。续跑后下一轮 Agent 按新规则生效。
- 审计对象 `target_kind`（`web` / `library` / `mixed`，默认 `web`）与挖掘模式、挖掘路径正交：描述「审计的是 Web 应用还是组件库」。创建时选定；暂停或完成后可改。规则与提示词在 `backend/app/target_kind.py`、`backend/app/prompts/target_kinds/`。组件库以公开 API / 解析器为入口，验证默认偏 harness 且通常关闭 Verifier；混合仓优先挖库核心、demo/sample 降权。Agent 不自行改写该字段。
- 挖掘路径与赏金/全量正交：`heuristic_enabled`（默认 true）/ `fast_enabled`（默认 false）/ `bypass_enabled`（默认 false）描述「怎么挖」。至少开一条；暂停或完成后可改。启发式在历史漏洞收集完毕后按文件挖。盖章时权重 100 / `MarkSource` 覆盖用户可控入口（HTTP 以及 WebSocket / RPC / MQ / 回调等，组件另含公开 API），不要只标 HTTP；Service / 过滤器 70–90，执行面 40–60，DTO/常量 10–30。Worker 按焦点角色挖：入口正向 source→sink，Service/过滤器回推或控面，低权薄扫。可开 `heuristic_lite`（默认 false）只把权重 100 的文件当入口，更低权重不阻塞完成。快速扫描 Recon 后 Semgrep → 代码筛（候选 200）→ SinkTriage → 冻结约 60 条 → Fast Worker 每轮 1 个 Sink 回推。历史漏洞绕过在历史漏洞收集完毕后把 `docs/old-vulns/` 文档冻结成队列，Bypass Worker 每轮注入 1 条尝试绕过补丁或确认未修复洞仍可打。`mining_complete` 须所有开启路径都结束且无 returned/fixing。SubmitVuln 按角色写入 `vulns.mining_path`（`heuristic` / `fast` / `bypass`），漏洞产出列表与详情展示对应中文标签。规则在 `backend/app/mining_paths.py`、`backend/app/services/sink_filter.py`、`backend/app/services/bypass_queue.py`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1diot9/VulnHunter-White](https://github.com/1diot9/VulnHunter-White) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
