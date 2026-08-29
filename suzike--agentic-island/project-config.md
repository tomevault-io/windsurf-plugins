---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 仓库现状（先读这一段）

这**已经不是设计交付包**，而是一个功能完整、可打包安装的 Windows 桌面应用 **Agentic-Island（灵动岛）**：常驻屏幕顶部的 AI 编码 Agent 监控/审批/协作面板 + 个人工作台。技术栈 **Electron 39 + React 19 + TypeScript**（electron-vite 构建，Vite 锁在 5.x）。已产出 NSIS 安装包（`dist/Agentic-Island-Setup-*.exe`）。

历史设计交付包不属于当前运行时源码；当前行为以 `src/`、`scripts/`、`docs/` 和自动化测试为准。

## 常用命令

```bash
npm run dev          # 开发运行（app ready 会自动安装全局 hooks；AIISLAND_SKIP_HOOKS=1 可跳过）
npm run typecheck    # 两套 tsconfig（node=main/preload+shared, web=renderer+shared）
npm test             # 顺序执行 36 个离线 test-*.ts（排除真实 Claude 登录探针）
npm run build        # electron-vite 三端构建
npm run package      # NSIS 安装包（原生模块经 asarUnpack **/*.node）
npm run verify:package # 隔离启动 unpacked、静默安装/启动/卸载 NSIS
npm run docs:capture # 隔离演示数据启动 Electron，重建 README 真实截图
npm run demo:plan    # 向运行中的岛注入一条演示计划（看 Plan 审阅长什么样）
npm run probe        # 诊断：一键接入+实时打印 hook 事件，Ctrl+C 还原

# 测试（raw node 直跑 TS，无测试框架）
node --experimental-strip-types scripts/test-lifecycle.ts   # hook→桥→状态机全生命周期 + 安装器幂等
node --experimental-strip-types scripts/test-loop.ts        # 审批阻塞闭环 + deny 理由回传
node --experimental-strip-types scripts/test-stop.ts        # git 变更小结（真实临时仓库）
node --experimental-strip-types scripts/test-codex-tail.ts  # Codex rollout 跟随
node --experimental-strip-types scripts/test-ics.ts         # ICS 解析（时区/RRULE/EXDATE）
node --experimental-strip-types scripts/test-notes.ts       # 便签 AI 解析器
node --experimental-strip-types scripts/test-quote.ts       # 问答上下文、分支与知识沉淀
node --experimental-strip-types scripts/test-methodologies.ts # 回答方法与气泡分析方法目录
node --experimental-strip-types scripts/test-providers.ts   # 模型目录迁移与供应商配置隔离
node --experimental-strip-types scripts/test-external-yield.ts # 外部应用让位
node --experimental-strip-types scripts/test-shortcuts.ts   # 快捷编排与安全闸
node --experimental-strip-types scripts/test-todo.ts        # 待办规划/统计/导入导出
node --experimental-strip-types scripts/test-recording.ts   # 录屏几何、时钟、字幕与 FFmpeg 参数
node --experimental-strip-types scripts/test-recording-session.ts # 分片落盘与异常恢复
node --experimental-strip-types scripts/test-recording-project.ts # v2 工程保存、复制与迁移
node --experimental-strip-types scripts/test-recording-export-e2e.ts # 真实 FFmpeg 导出
node --experimental-strip-types scripts/test-terminal-workspace.ts # 终端现场迁移、加密与脱敏
node --experimental-strip-types scripts/test-terminal-project.ts # 项目任务发现与健康检查
npm run audit:terminal # 隔离 Electron 验证输入、退出码、危险确认与恢复
npm run audit:ask      # 隔离 Electron 验证回答方法、气泡分析与窄宽布局
```

## 架构总览

```
Claude Code (CLI/桌面端) ──hooks──► src/hooks-bin/cc-forward.mjs ──HTTP 127.0.0.1──►┐
Codex (CLI/桌面端) ────rollout 日志──► src/main/codex-tail.ts（轮询跟随）──────────────►│
                                                                                    ▼
              ┌────────────────────── Electron 主进程 (src/main) ──────────────────────┐
              │ bridge-server  本地桥(随机端口+token→~/.agentic-island/bridge.json,    │
              │                15s 自愈防覆盖)；permission 事件阻塞至用户裁决           │
              │ agents-store   Agent 状态机(会话键=backend:sessionId；SessionEnd 即移除)│
              │ hook-installer 合并式安装/卸载 ~/.claude/settings.json + ~/.codex hooks │
              │ term-pty       真 PTY 终端(@lydell/node-pty N-API, ConPTY, 多会话)      │
              │ calendar-ics / calendar-caldav  飞书日历(CalDAV multiget 两段式)        │
              │ rss / media(SMTC) / clipboard-watch / sound / llm-proxy / git-summary  │
              │ recording-session / project / export  分片落盘、工程与 FFmpeg 导出    │
              │ terminal-jump(HWND/UIA 切 WT 标签页) / settings-store(DPAPI 加密)      │
              └───────────────────────────┬── Electron IPC ────────────────────────────┘
                                          ▼
              渲染进程 (src/renderer)：App.tsx 编排 + components/*Tab + logic/* + ui/*
              十一个分区：Agents · Plan · 问答 · 快捷 · 待办 · 灵感便签 · 资讯 · 复盘 · 仓库 · 终端 · 设置
              + AmbientBar 常驻迷你条（收起后的小状态条，多模式轮播）
```

- **设计系统（src/renderer/src/ui/）**：2026-07 前端视觉全面重设计 + Apple（macOS/iOS）化后，**所有新 UI 必须走这里**：
  - `tokens.ts`：**填充制层级**——`fill(1-4)` 填充阶梯代替 rgba 白透明度与 1px 描边、`hairline()` 0.5px 发型线分隔、`separatorRow()` 分隔行、`R` Apple 圆角阶梯（按钮 10/卡片 13/浮层 18/面板 28）、`ink(1-4)` iOS label 四级墨色、`text.*` SF 排版（标题负字距）、`accent()/sem.*` 语义色（全部消费 OKLCH 主题变量，勿手写 oklch 魔法值）。
  - `components.tsx`：共享组件 Button（含 tinted 变体）/IconButton/Card/Chip/Badge/Input/Segmented（iOS 滑动 thumb）/SectionHeader/EmptyState/Switch（iOS 白钮）/Slider/Group（inset grouped 列表）。
  - `motion.ts`：framer-motion 预设（fadeScaleIn 卡片入场 / overlayPop 浮层 / stagger 列表 / pressable iOS 透明度下沉）。
  - `icons.ts`：lucide 语义图标表 `Ico`（全岛 emoji 图标已清除；**用户数据里的 emoji 字段保持渲染**）。
  - 参考样板：`components/AgentsTab.tsx`。

- **协议契约**：`src/shared/protocol.ts`（三端共用；`IslandBridgeApi` 是 preload 暴露的全部能力面）。
- **Claude Code 接入**：全生命周期 hooks（SessionStart/UserPromptSubmit/PreToolUse/Stop/Notification/SessionEnd）。PreToolUse 对非只读工具**阻塞审批**（stdout 返回 permissionDecision）；deny 理由回传实现接力 steer；ExitPlanMode→计划审阅；Stop→"等待回复"+ transcript 尾部提取最后回复 + turnEnd 触发 git 小结。
- **Codex 接入**：rollout 日志跟随为主（`~/.codex/sessions/**/rollout-*.jsonl`，只监控无审批，15min 空闲自动归档）；hooks 桌面端实测会触发（审批可用），CLI 不触发。**CLI/桌面端无法区分**（originator 恒为 codex-tui），统一标 "Codex"。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suzike/agentic-island](https://github.com/suzike/agentic-island) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
