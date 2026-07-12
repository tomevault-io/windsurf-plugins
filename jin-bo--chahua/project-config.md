---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目语义

「茶话室」(chahua) = 多 Agent 群聊桌面 App。用户和多个由 [agentao](https://github.com/jin-bo/agentao) 驱动的 AI「茶客」在同一聊天室对话（像微信群，可 `@`、茶客之间也能接话）。房间不只是聊天容器 —— 还是带任务的工作容器（**P5 任务房间**：开任务 / propose 决策待用户采纳 / `./task/<name>` 自动归集 artifact / 多任务共存单时刻 1 active）+ 自带取证视图（**P6 调试与回放**：每轮可展开看候选 / 分数 / prompt / 工具 / 产物，落盘 `rooms/<id>/debug/`）。

**两个运行形态共享同一套 Python 后端**，都走 `chahua.session.build_room_session()` 装配房间：
- CLI REPL：`uv run chahua`（最快验证 LLM 凭据 / `room.toml`）
- Electron 桌面壳：`cd app && npm run dev` —— main 进程拉起 `chahua-server` sidecar，本地 WebSocket 通信

## 常用命令

```bash
uv sync                                       # Python 依赖（按 pyproject.toml 拉 agentao≥0.4.14）
uv run chahua                                 # CLI（默认入 rooms/p1-test）
uv run chahua --room rooms/p3-黄河路
uv run chahua-server --host 127.0.0.1 --port 7860 --room rooms/p3-黄河路  # 独跑 sidecar
uv run pytest                                 # 全量测试（~45s）
uv run pytest tests/test_xxx.py -v            # 单测
cd app && npm run dev                         # Electron dev（首次需 npm install）
cd app && npm run build:python && npm run build:mac  # 打包 → dist/茶话室-<ver>-mac-arm64.dmg
FORCE=1 npm run build:python                  # 强制清重建 python-bundle
```

`pyproject.toml` 设 `asyncio_mode = "auto"` —— async 测试无需 `@pytest.mark.asyncio`。

## 架构要点

### 双层进程 / 双层路径

```
Electron main (Node)  ─ spawn ─→  chahua-server (Python sidecar)
       │                                │
       └─ stdio: ["pipe", ...]           └─ WebSocket 127.0.0.1:<random-free-port>
       └─ before-quit → stop()           └─ stdin EOF watcher → graceful 关停
```

- **app_root vs user_data_root**：dev 同源仓库根；packaged app_root=`.app/Contents/Resources`、user_data_root=`~/Library/Application Support/chahua/`。Electron export `CHAHUA_APP_ROOT` / `CHAHUA_USER_DATA`，Python `Paths.from_env()` 接住。**改路径解析双根都顾**（persona 走 `find_in_data_then_app`：user_data 优先回退 app）。
- **sidecar ready 信号**：`server.py` 打 `监听 ws://...`，`sidecar.js` 正则 `/监听\s+ws:\/\//` 匹配后 resolve。
- **首启动 seed**：dev 跳过；packaged 拷 `app/templates/` → user_data_root，`.chahua-seeded` 幂等。
- **跨平台 sidecar 关停**：`sidecar.js::stop()` 先 `child.stdin.end()` → Python `_watch_stdin_eof` 收 EOF graceful；2s grace 兜底 `forceKillTree`（Windows `taskkill /F /T`，POSIX `SIGKILL`）。Windows `connect_read_pipe(sys.stdin)` 常 `WinError 6` 静默失败，tree-kill 是兜底正解。

### Python 后端模块分工

职责一句话；承重契约见「关键不变量」段，实现细节看代码。

- `session.py` — 房间装配。CLI 与 server 共用 `build_room_session()` / `discover_rooms()` / `load_env_files()`。
- `config.py` — `room.toml` 解析，白名单严格（未知字段 `RoomConfigError`）。字段：`[room]` / `[room.llm]` / `[scoring]` / `[summary]` / `[[guest]]` / `[[guest.extra_mcp_servers]]`。
- `llm_spec.py` — `LLMSpec` 三入口（`from_env` / `try_from_env` / `from_toml`）+ `build_client()`。toml 强制 `model="<provider>/<model>"`，env 允许裸 model。各 section 走自己 spec，缺即 fallback 回房间默认。
- `guest.py` — `TeaGuest`（包 `agentao.Agentao`）。`speak()` 外层 try/except/finally 保 `message_start`/`message_end` 成对，envelope 与 transcript 共享 message_id。P13 视觉：`speak(images_rel=())` → `resolve_images` → `arun(images=...)`，懒读现传。
- `image_input.py` — P13 视觉 helper（server inbound 与 guest 共用）。`_normalize_share_image_rel`（纯校验）+ `resolve_images`（IO：normalize → symlink 围栏 → 读 bytes → base64+MIME）。限额 `from agentao.media_limits`。
- `orchestrator.py` + `_orchestrator_{chain,handoff_drain,handoff_queue,managed_session,scoring,consts}.py` — 意愿打分主循环：并发打分 → 取 ≥ `want_threshold` 前 1~2 名发言 → 无人过阈值等用户；`@<名字>` 确定性路由不打分。5 个 slot 经 `_install_orchestrator_slots(orch)` 单点装配，公开 import 路径不动；主类保留 `_run_ai_chain` / `_intercept_task_proposal` 同名 method（供 monkeypatch / hook 取 bound method）。
- `scoring.py` — 轻量打分。transcript 不可信，严格 JSON、解析失败降级 0、score clamp `[0,1]`。
- `mentions.py` + `orchestrator_config.py` — 前者 `@` 提及解析（broadcast token / 白名单边界防 URL·email 误配 / 全 Unicode 空白）；后者编排参数 frozen dataclass（拆 context_renderer↔orchestrator 循环 import）。均从 orchestrator 抽出，原 import 路径 re-export 不动。
- `room.py` + `cursor.py` + `_persist.py` — 持久化层。`transcript.jsonl` / `summary.jsonl` append-only 加载跳坏行；`cursor.json` tmp+rename。不做 fsync。
- `events.py` — `ChahuaEnvelope`（含 `schema_version`）。茶话室外层合成 room_id / guest_name / message_id（agentao 原生 `AgentEvent` 不带）。
- `transport_bridge.py` — `ChahuaTransport`（`SdkTransport` 子类），agentao 事件 → envelope；P10 artifact 反查 `_maybe_record_artifact_path`。
- `summarizer.py` — cheap LLM 增量产 `summary.jsonl`，onboarding 窗口拼用。
- `permissions.py` — read-only 双 API 同步：统一 `apply_permission_mode(agent, mode_str)`，别处禁单调 `set_mode`。
- `task.py` + `tasks_store.py` — 任务数据模型 + `tasks/state.json` + `tasks/<id>/{task.json, decisions.jsonl, artifacts/}`。`build_task_info_payload` 是 `task_info` 投影单一来源。入站严格 / 落盘宽容。
- `task_tools.py` — 5 个茶客侧 task 工具：`task_list_artifacts`（真无副作用）/ `task_propose_{decision,open,status}`（`is_read_only=True` 但 emit `TASK_PROPOSAL`）/ `task_write_artifact`（绕 `PathPolicy` 让 `./task/<name>` 落盘，`is_read_only=False`；name 拒 `/` `\` `..` `.` 前缀）。
- `handoff_tools.py` — 3 个茶客侧 handoff propose：`propose_delegate` / `propose_review` / `propose_panel`。不带 `task_` 前缀（房间级调度）。`propose_review` 把 `reviewee` 名解析成最近发言 `message_id` 冻结。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jin-bo/chahua](https://github.com/jin-bo/chahua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
