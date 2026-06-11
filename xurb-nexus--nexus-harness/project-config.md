---
trigger: always_on
description: > 三端通用入口（Codex / Claude Code / Cursor）。所有 AI Agent 进入本仓库后，**第一件事是读这个文件**，然后按指引找 skill。
---

# nexus-harness · Agent 入口

> 三端通用入口（Codex / Claude Code / Cursor）。所有 AI Agent 进入本仓库后，**第一件事是读这个文件**，然后按指引找 skill。

---

## 启动入口引导

首次进入本仓库时，先判断用户首条输入是否以 `开发者模式` 或 `启动开发者模式` 开头。若命中，视为当前会话在维护 nexus-harness 框架本身：不展示 `[agent.md](./agent.md)` 启动入口，不做 `1 / 2 / 3 / 4` 路由，直接回答用户在该输入中的问题或按其维护目标处理。

**命中 `开发者模式` 时第一件事**：在你给用户的**首条**回复中包含一行 HTML 注释 `<!-- NEXUS:DEV_MODE -->`（用户看不见，但 `.cursor/hooks/agent_response_audit.py` 会捕获并写入 `.nexus/conversation/<conversation_id>/dev_mode`，让本 Cursor 实例的 stop / subagent / audit 钩子全部短路放行）。这个标记是**per-conversation** 的，只影响本 Cursor 实例，不会污染同时打开的其他 Cursor 实例。

新协议（per-conversation）和旧协议（仓库级 sentinel）共存：

- **新协议优先**：宿主 stdin 携带 `conversation_id` 时，stop hook 用 `.nexus/conversation/<id>/` 子目录的 `dev_mode` 标记 + `plan_dir` 记录决定是否干预；没记录的会话一律 ALLOW。
- **旧协议作 fallback**：宿主无 `conversation_id` 时，仍读 `.nexus/hook_paused` 全局 sentinel；当下游业务 active session 出现"被另一个 Cursor 维护会话误激活"时也可手动 `mkdir -p .nexus && touch .nexus/hook_paused` 紧急压制（重启会话即恢复）。
- 两份状态都不入 git；`.nexus/` 已在 `.gitignore`。

stop hook 引入 per-conversation `dev_mode` 标记 + 全局 `DEVELOPER_SESSION.json` 解析：默认按 workspace 中最近活跃的 `DEVELOPER_SESSION.json` 决定要不要拦；当前会话若有 `<!-- NEXUS:DEV_MODE -->` 标记（写到 `.nexus/conversation/<id>/dev_mode`），stop hook 直接放行——这是同时开两个 Cursor 实例时维护会话不被拉的唯一隔离手段。

未命中开发者模式时，展示 `[agent.md](./agent.md)` 的启动入口内容：`nexus harness` 字符横幅和 4 个一级选项。

非 CLI 宿主（如 Cursor 桌面版、VS Code AI 对话框、JetBrains/IDEA AI 对话框）无法像 Claude Code CLI 一样在空白启动页自动显示公告。因此在这些宿主里，**首次收到用户任何输入时**，除非命中开发者模式，否则必须先展示 `agent.md` 启动入口，再处理用户意图；如果用户输入或明确提到 `nexus` / `Agent` / `启动入口`，也必须重新展示该入口。

非 CLI 宿主展示入口时，只原样展示 `agent.md` 中的启动页代码块；禁止在代码块下方重复列出 `1 / 2 / 3 / 4`，禁止追加第二段“你想做什么”。`agent.md` 已经包含“下次输入 nexus 或 Agent 可回到此入口”的提示，不需要额外补充。

用户回复编号或自然语言意图后，按以下规则路由：

- `**下一步` / `继续` / `接下来` / `不知道怎么走` / `往下走` / `卡住了` / `然后呢` / `迷失了` 等恢复意图** →
 运行 `python3 scripts/what_next.py`（**会话隔离已内建**：脚本自动按 `.nexus/conversation/<id>/{plan_dir,workspace_dir}` 绑定锁定本会话项目，多 Cursor 实例并发时不会被另一个实例的 active session 吸走；conv_id 由 `.cursor/hooks/shell_plan_dir_bind.py` 通过 sidecar 自动注入，AI 无需手动传 `--conversation-id`）。按返回结果处理：
  - `mode=single_project`：
    - **若返回字段 `auto_advance=true`**（高置信单项目 + 当前 `next_required_action` 不属于 `user_review_gate / none / pause / handle_blocker`）：直接展示 `user_message`（一行进度）+ **立刻**按 `agent_instruction` 执行，**不等用户回复**。后续由对应 skill（典型为 Developer 的 `developer_autopilot.py`）接管。Cursor / Claude Code 若已挂 `.cursor/hooks.json` / `.claude/settings.json` 的 stop hook，Agent 试图擅自停顿时会被自动拽回循环；hook 不可用时仍以本规则为准。
    - **否则（`auto_advance=false`）**：把 `user_message` 原样展示给用户，等用户选 1/2/3；
    用户选 `1` → 按 `agent_instruction` 直接进入对应 skill，**禁止再走 `transition_confirmation` 或重新展示项目列表**；
    用户选 `2`/`3` → 按字面意思处理。
  - `mode=multi_project`：把 `user_message` 原样展示给用户，等用户选项目编号；
  用户选编号 N → 运行 `python3 scripts/what_next.py --select N`，得到 `mode=single_project` 后按上面规则处理，**禁止再走 `transition_confirmation`**。
  - `mode=no_projects`：把 `user_message` 原样展示给用户，等用户选 1/2/3。
  - **禁止把 `what_next.py` 的输出再走 `transition_confirmation` 或 `discover_workspace_stage.py`**。
- ##### `1` / 开始新项目 / 新需求 / 我要开始 → 读取 `skills/project-start/SKILL.md`，进入 `project-start`。
- `2` / 继续项目 / 恢复项目 / 继续开发 → 先运行 `python3 scripts/discover_workspace_stage.py` 做 workspace 阶段识别；该脚本会静默把超过 30 天未更新的进行中 session 标记为 `stale`（不删除内容），再按识别结果进入对应 skill。
- `3` / 提取 AI 知识库 / 构建 docs / 更新 docs / build aiweave / sync aiweave → **立即** Read `skills/docs-build/SKILL.md`，由该 skill 接管后续全部交互。**禁止**先列 `skills/` 子目录做 skill 发现；旧版 `knowledge-extract` / `knowledge-extract-reviewer` / `knowledge-qa-reviewer` 已物理下沉到 `skills/_deprecated/`，**仅作 git 历史留档**，任何场景都不得读取或调用其中文件、脚本。收到 3 / 提取 AI 知识库意图后，下一个动作只能是 Read `skills/docs-build/SKILL.md`，无任何中间步骤。
- `4` / 归档项目 / 项目归档 / 项目收尾 / 项目完结 / finish project → 读取 `skills/finish-project/SKILL.md`，进入 `finish-project`。
- `5` / 问题定位 / 联调 / 排障 / 自测发现 / 联调发现 / 线上问题 / 线上故障 / diagnose → 读取 `skills/diagnose/SKILL.md`，进入 `diagnose`。
- `6` / 查看使用说明 / 打开文档 / 文档站 / 帮助 / docs / help → **宿主区分**：（**A**）**Cursor 桌面版 / Cursor Agent 聊天 / 其它无法在子 shell 可靠调用系统 `open` 的环境**：**禁止**执行 `bash .claude/hooks/start-docs.sh`。用工作区仓库根拼出 `site/public/index.html` 绝对路径，回复中**最多两行**：① 可点击的 Markdown 链接 `[file://绝对路径](file://绝对路径)`（**禁止**粗体包裹 URL）；② 一句「若无法点开，请复制 file:// 地址到浏览器地址栏」。**不要**把脚本整段输出粘贴进聊天。**（B）Claude Code CLI 等本机终端环境**：运行 `bash .claude/hooks/start-docs.sh`。脚本只做以 `file://` 打开本地 `index.html`，零运行时、无后台进程；stdout 原样展示。脚本退出后再用 Markdown 链接格式输出其打印的 `URL` 行。**禁止**用粗体包裹 URL。回到入口等待下一步，**不要追加**推荐下游 skill。

入口只负责一级选择；一旦命中具体 skill，必须立即读取该 skill 的 `SKILL.md` 并完全交给该 skill 接管输出格式、ASK 节点和后续状态机。入口层禁止二次包装下游结果，禁止自行模拟下游菜单，禁止把入口判断摘要和下游 skill 的项目选择/任务确认混在同一段里。

**全局返回入口（覆盖所有 skill 的 ASK / 等待输入）**：无论当前处在哪一个 skill 的状态机、是否在等待编号 / 是 / 否 / 确认文案，只要用户本条输入**仅为** `nexus` 或 `Agent`（单独一行；大小写不敏感；无其它实质指令），主 Agent 必须**立即退出当前 skill 的追问语境**，按 `.cursor/rules/agentflow.mdc` 与上文「非 CLI 宿主」规则**重新展示 1–6 启动入口**。**禁止**把 `nexus` / `Agent` 当作「无效选项 / 请重新回复 1」处理；**禁止**把该字符串当作 `selection` / `confirmed` 等参数继续调用当前 skill 的 wizard。用户在新入口重新选 `1`–`6`（或自然语言映射）后，再按路由进入对应 skill。

特别是选择 `2` 时：

- 入口层只允许运行 `scripts/discover_workspace_stage.py` 做轻量阶段识别与 session 过期静默标记；不要手写扫描逻辑、不要自行推断阶段。
- 如果脚本识别到 `stage=project_start_in_progress` / `route_skill=project-start`，立即读取 `skills/project-start/SKILL.md`，用脚本返回的 `session_path` 恢复 project-start。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xurb-nexus/nexus-harness](https://github.com/xurb-nexus/nexus-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
