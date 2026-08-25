---
trigger: always_on
description: > 目的:把本项目的架构、协议、各后端调用方式、**所有踩过的坑(含原因)**、扩展方法固化下来,
---

# AGENTS.md — 给在本仓库工作的 AI agent 的工作手册

> 目的:把本项目的架构、协议、各后端调用方式、**所有踩过的坑(含原因)**、扩展方法固化下来,
> 让上下文压缩 / 新会话后仍能无损接续。配套 `plan.md`(进度)、`README.md`(用法)。
> 环境:Windows 11,MATLAB R2025b(`D:\Software\Matlab2025b`),Node v25,bash(Git Bash)。

## 0. 一句话

MATLAB 内嵌侧边栏 AI 助手:`uihtml` 面板 ⇄ MATLAB ⇄ 本地 Node sidecar ⇄ headless 后端(Claude Code / Codex)⇄ 复用的 MATLAB MCP。

## 1. 数据流(必须牢记)

```
JS(ui/index.html)
  │  sendEventToMATLAB(name, data)              JS → MATLAB
  ▼
Panel.onHtmlEvent(evt)  → Bridge.send(struct)   组装上下文后下发
  │  tcpclient writeline(asciiJson(jsonencode))  MATLAB → sidecar(全 ASCII 行 JSON)
  ▼
server.handleClientLine → adapter.sendMessage()
  │  spawn 后端 CLI,stdout 行 → 翻译成 OutMsg 事件
  ▼
adapter 'event' → server.toClient → tcpclient    sidecar → MATLAB
  │  Bridge.onData 排空所有行 → Panel.onSidecarMessage → pushToUi
  ▼
sendEventToHTMLSource(HTML,'sidecar', asciiJsonString)  MATLAB → JS(传字符串!)
  │  JS onSidecar(JSON.parse) → 渲染
```

权限走**另一条** TCP(控制端口):后端的 `--permission-prompt-tool` MCP(`permissionServer.js`)回连 sidecar 控制端口 → 只读自动放行 / 破坏性转 UI 确认卡 → 用户点击 → 回传 decision。

## 2. 端口 / 文件

- 首选端口:client `8765`、control `8766`(env `MATLAB_SIMULINK_SUPERAGENT_PORT` / `..._CONTROL_PORT`)；`Bridge.start` 启动前检测冲突，默认自动选择空闲端口对，`AutoSelectPorts=false` 才严格失败。
- 发现文件:`%TEMP%/matlab-simulink-superagent-sidecar.json`(pid/端口/backend)。
- 日志:`%TEMP%/matlab-simulink-superagent-sidecar.log`、各后端自己的日志。
- 启动成功标志:sidecar 向 stdout 打 `SIDECAR_READY {...}`(Bridge 解析它判断就绪)。

## 3. 协议(sidecar/src/protocol.js)

**InMsg(UI/MATLAB → sidecar)**:`user_message`(含 context、可选 intent=insert_at_cursor、可选完整 config)、`interrupt`、`permission_response`、`transaction_ready{id,ready}`、`ping`、`set_config{config}`、`get_capabilities`、`slash_command{name,args,context,config?}`、`close_conv`、`change_recorder_control{action,projectRoot?,task?}`(action 含 start/stop/status/configure/approve/execute/validate/export)、`change_recorder_entry{entry}`、`change_recorder_enrich{id,sequence,semantic}`。**多会话**:大多 InMsg 带 `convId`(标签页/分支 id,缺省为 `main`);工程记录器事件是跨会话的工程状态。`permission_request`/`permission_response` 带 convId;权限 pending 用 `convId::id` 复合键。新标签/Fork/隐藏体检会话的首条消息必须携带 UI 继承的完整 config，让 sidecar 在创建 adapter 前原子应用。MATLAB-only 事件另加 `copy_text`(走 `clipboard('copy')`)。

**OutMsg(sidecar → UI)**:`ready`、`status`、`thinking_start/delta/stop`、`plan_update{id,steps,explanation?}`、`assistant_start/delta/stop`、`tool_use`、`tool_result`、`permission_request`、`transaction_prepare{id,tool,input}`、`result`、`error`、`pong`、`capabilities`、`config_changed`、`audit{entry}`、`change_recorder_state{state}`、`project_change{entry}`、`change_report{report}`。

**MATLAB 直接推给 UI 的事件**(不经 sidecar,`Panel.pushToUi`):`context`、`theme{mode}`、`user_echo{text}`、`attachments{files}`、`diagnostics{items}`(结构化诊断卡片)、`change_transaction`(模型检查点/验证/回退结果)、`mbse_workflow_state{state}`、`status`、`error`。

**仅 MATLAB 处理的 UI 事件**(不下发 sidecar):`ui_ready`、`diagnose_error`(结构化诊断)、`ask_at_cursor`、`request_context`、`request_theme`、`find_block{query}`(🔍查找模块)、`explain_selected`(🧩解释模块)、`jump_block{path}`(诊断卡片跳转高亮)、`run_tasks`(⚙ 自适应任务流)、`self_heal`(🔄自愈环)、`capture_model`(📸截图分析)、`requirements`(📋需求追溯)、`codegen_review`(🔬代码评审)、`batch_edit{query}`(🪄批量编辑)、`test_orchestrate`(🧪▶测试编排)、`attach_file`(任意类型;图片→视觉)、`attach_image{name,dataUrl}`(粘贴图片:base64→临时 PNG)、`copy_text`、`close_conv`、`clear_attachments`、`remove_attachment{index}`。附件 struct 统一 `{name,content,path,isImage}`:文本嵌 content,图片给 path 让 agent 用 Read 读图(preamble 在 `types.js`)。这些 MBD 快捷动作的 handler 均走同一模式(采集 context → 构造 prompt → `Bridge.send(user_message)`);`onHtmlEvent` 顶层 try/catch 兜底:出错发 `error` 事件解除 UI busy + 复位 `PendingFind/PendingInsert`。

MBSE 例外仍是 MATLAB 本地事件：UI 发 `mbse_workflow{action,phase,projectRoot,workflowConfig}`，`Panel` 直接调用 `MBSEWorkflow`，不进入 sidecar 后端协议；状态以 `mbse_workflow_state` 回 UI，工程变更条目另送 `change_recorder_entry`。

序列化必须用 `serialize()`(把非 ASCII 转 `\uXXXX`)。行缓冲 `createLineBuffer`。

## 4. 后端适配器(sidecar/src/adapters/)

基类 `BackendAdapter`(EventEmitter):`start/sendMessage/interrupt/stop`,子类 `emitEvent(OutMsg)`。
`makeAdapter(state)` 在 `index.js`,按 `state={backend,model,effort,mode}` 造适配器;`Server.applyConfig` 运行时重建。

### ClaudeCodeAdapter
- 命令:`claude --print --output-format stream-json --include-partial-messages --verbose`
  - `--resume <session_id>`(多轮)、`--model`、`--allowedTools`(预批只读)、
  - `--permission-mode <acceptEdits|plan>`(ask=不加)、`--mcp-config <file> --strict-mcp-config`、
  - `--permission-prompt-tool mcp__approval__approval`、`--append-system-prompt`。
- MCP 配置文件由 `index.js` 生成:**只含 `approval` + `matlab`**(`--strict-mcp-config` 防止按 cwd 加载到无关项目级 MCP)。
- thinking:`buildEnv` 设 `MAX_THINKING_TOKENS`(effort→token:low/med/high=2000/8000/16000)。
- 流式:`streamJsonParser.js` 把 `stream_event`(message_start / content_block_start/delta/stop / message_stop)翻译;text_delta→assistant、thinking_delta→thinking;ASSISTANT_START **懒发**(首个 text_delta 才发,避免纯思考/工具消息留空气泡)。
- **常驻模式(opt-in,`MATLAB_SIMULINK_SUPERAGENT_PERSISTENT=1` 或 UI ⚡ 开关 → `persistent: true`)**:`start` 预热常驻进程(`--input-format stream-json`),每轮往 stdin 写一行 `{type:'user',message:{...}}`(不关 stdin),跨轮复用一个 translator;消除每轮冷启。**故障安全**:进程意外退出 → 下轮以 `sessionId` resume 重启;中断 → kill+resume;`resetSession`(/compact)→ kill 换新 session;per-turn systemPrompt(insert)→ 指令并入消息文本(常驻无法热换 system prompt)。默认仍走可靠的 resume 模式(`sendOneShot`)。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suzike/Matlab-Simulink-SuperAgent](https://github.com/suzike/Matlab-Simulink-SuperAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
