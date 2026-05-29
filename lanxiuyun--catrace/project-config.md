---
trigger: always_on
description: Catrace 是一款桌面端工具，帮助用户平衡工作与休息。
---

# Catrace — Agent Guide

> 本文档面向 AI 编程助手。

---

## 项目概述

Catrace 是一款桌面端工具，帮助用户平衡工作与休息。

- **核心功能**：后台静默监听键鼠活动，判断用户是否处于连续工作状态；当连续活跃时间超过阈值时，通过系统通知提醒用户休息。
- **隐私承诺**：不偷拍屏幕、不上传数据，所有信息保存在用户本地。
- **当前状态**：**已实现核心功能**，前端 Dashboard 可查看今日活动与统计，Rust 后端已完成采样、判定、通知、数据库全流程。

---

## 仓库现状

```
.
├── README.md
├── PLAN.md
├── AGENTS.md
├── package.json          # pnpm + Vite + Vue 3
├── vite.config.ts
├── tsconfig.json
├── index.html
├── src/                  # Vue 3 前端
│   ├── api/tauri.ts
│   ├── assets/
│   ├── components/
│   │   ├── Timeline.vue        # 详细视图：24h 分钟级色块热力图（CSS Grid）
│   │   └── TimelineWindows.vue # 概览视图：block 卡片网格（可展开整行）
│   ├── router/index.ts
│   ├── utils/
│   │   └── timeBlocks.ts       # 前瞻式 block 切分（前后端共用逻辑）
│   ├── views/
│   │   ├── Dashboard.vue
│   │   ├── Settings.vue
│   │   └── Debug.vue
│   ├── App.vue                 # 布局 + naive-ui 主题注入
│   ├── theme.ts                # 统一色板 + naive-ui themeOverrides
│   ├── main.ts
│   └── vite-env.d.ts
├── src-tauri/            # Tauri 2 + Rust
│   ├── src/
│   │   ├── main.rs       # 入口，调用 lib::run()
│   │   ├── lib.rs        # 全部业务逻辑（采样、结算、通知、命令）
│   │   └── db.rs         # rusqlite 封装
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── ...
└── public/
```

> **注意**：Rust 侧未按 PLAN.md 的分层目录（`input/`、`engine/` 等）实现，而是将所有逻辑集中在 `lib.rs` 中，通过模块级函数组织。

---

## 已落地的技术栈

| 层级 | 选型 |
|------|------|
| 桌面框架 | Tauri 2 |
| 前端 | Vue 3 + TypeScript + Vite + naive-ui |
| 图表 | **未使用 ECharts**（时间轴用 CSS Grid 实现） |
| 后端（Rust）| rdev（键盘）、device_query（鼠标）、rusqlite（DB）、tokio、active-win-pos-rs（焦点窗口）、tauri-plugin-autostart、tauri-plugin-opener、tauri-winrt-notification（Windows Toast）、windows-registry |

---

## 核心逻辑（已实现）

1. **采样**（`lib.rs`）
   - 每 2 秒检查鼠标光标位置（`device_query`）。
   - 全局监听键盘按下事件（`rdev`），2 秒内去重。
2. **分钟判定**（`lib.rs`）
   - 60 秒内活动次数 ≥ 3 → 该分钟标记为**活跃**；否则标记为**休息**。
   - **视频/流媒体检测**：若键鼠活动不足，但检测到正在播放视频，该分钟仍视为**活跃**。
     - **Windows**：优先尝试 `GlobalSystemMediaTransportControlsSessionManager` 枚举系统媒体会话，只要有会话处于 **Playing** 状态即算活跃（不限 `PlaybackType`，覆盖浏览器、UWP 播放器、Spotify 等）。GSMTCSM API 调用成功时完全信任其结果（无 Playing 会话也视为不活跃），仅在 API 调用失败时才回退到窗口标题 + 进程名关键词匹配。
     - **macOS / Linux**：直接走窗口标题 + 进程名关键词匹配（YouTube、Bilibili、Netflix、VLC 等），基于 `active-win-pos-rs`。
3. **Block 切分与提醒**（`db.rs` + `lib.rs` + `utils/timeBlocks.ts`）
   - 从首个有记录的时间点开始，向后以 `window_minutes` 为单元切分 block：
     - 若在窗口内遇到连续 `break_minutes` 休息 → 切为**休息 block**（到连续休息结束）。
     - 若窗口内无足够连续休息 → 切为**活跃 block**（固定 `window_minutes` 长度）。
   - **关键约束**：切分只考虑「已发生的分钟」（索引 ≤ `nowIdx`）。未来未记录的 `null` 不会被当作「连续休息」来结束当前 block，避免切出从当前时间直通午夜的幽灵休息 block。
   - 当前时间所在为未完结的「进行中 block」。
   - 提醒逻辑（`db.rs` 切分 block + `lib.rs` 触发通知）：
     - 前一个已完成 block 为**活跃** → 弹出提醒（刚干完一波）。
     - 前一个已完成 block 为**休息**，当前进行中 block 长度 ≥ `window_minutes` → 弹出提醒（休息后又工作满一波）。
     - 其余情况不提醒。
   - 通知**不去做重**：只要条件满足，每分钟结算都会弹，直到用户连续休息够 `break_minutes`。
   - **休息即静音**：只要当前分钟在休息（无论是否达到 `break_minutes`），立即不提醒；恢复活跃后重新判断。
   - **Toast 按钮操作**（Windows）：通知带三个按钮——「3分钟后提醒」「5分钟后提醒」「跳过本次」。点击后直接更新 `ReminderState`，无需打开主窗口。

**提醒场景示例（`window=45, break=5`）**

> 关键前提：提醒只在**当前分钟活跃**时检查。休息分钟不检查，因此休息期间**绝不弹通知**。

| 场景 | 时间线 | 结果 |
|---|---|---|
| 活跃 45min → 继续活跃 | 0:00~0:44 活跃 → **0:45 弹** → 0:45~0:54 继续活跃（每分钟催） | ✅ 提醒 |
| 活跃 45min → 休息 1min → 继续活跃 | 0:00~0:44 活跃 → 0:45 休息（不催）→ **0:46 弹** → 0:46~ 活跃（每分钟催） | ✅ 提醒（休息即停，复工又催） |
| 活跃 45min → 休息 4min → 恢复活跃 | 0:00~0:44 活跃 → 0:45~0:48 休息（不催）→ **0:49 弹** → 0:49~ 活跃（每分钟催） | ✅ 提醒（休息不够，复工即催） |
| 活跃 45min → 休息够 5min | 0:00~0:44 活跃 → 0:45~0:49 休息（不催）→ 0:50 休息够 5min | ❌ 不提醒。休息期间不检查；休息够后 should_notify=false，恢复活跃需再工作满窗口 |
| 活跃 45min → 休息 5min → 再活跃 45min | 0:00~0:44 活跃 → 0:45~0:49 休息（不催）→ 0:50~1:34 活跃 → **1:35 弹** → 1:35 后每分钟催 | ✅ 提醒 |
| 活跃 40min，进行中 | 0:00~0:39 活跃（未满窗口） | ❌ 不提醒 |
| 活跃 40min → 休息 5min → 再活跃中 | 0:00~0:39 活跃 → 0:40~0:44 休息 → 0:45~0:47 活跃（未满窗口） | ❌ 不提醒 |
| 全天休息 | 一直在休息 | ❌ 不提醒 |

> 规律：活跃 block 完成后，**下一个活跃分钟**会弹；若之后没有休息够 `break_minutes`，则继续每分钟催。但只要**当前分钟在休息**，立即停止提醒；恢复活跃后重新判断。

---

## 配置项

| 配置名 | 说明 | 默认值 |
|--------|------|--------|
| `window_minutes` | 工作窗口长度（分钟） | 45 |
| `break_minutes` | 连续休息多少分钟算断开（分钟） | 5 |
| `silent_start` | 开机自启时不显示主窗口 | false |
| `video_active_enabled` | 视频计入活跃（开启后看视频算活跃，活跃时长到达后仍会提醒休息） | true |
| `locale` | 界面语言（zh-CN / en-US） | 自动检测系统语言，回退 zh-CN |

**提醒操作（进程级状态，重启后重置）**

| 操作 | 效果 |
|------|------|
| 跳过本次 | 当前 block 完成前不再提醒 |
| 3分钟后提醒 | 推迟 3 分钟，期间不弹通知 |
| 5分钟后提醒 | 推迟 5 分钟，期间不弹通知 |

> 只要当前分钟在休息，系统**自动不提醒**，同时清除 snooze。恢复活跃后重新判断。


---

## 实际目录结构

### Rust 后端（Tauri 侧）

```
src-tauri/src/
├── main.rs    -- Tauri 入口，仅调用 lib::run()
├── lib.rs     -- 全部业务逻辑：
│                · 键盘/鼠标采样线程
│                · 每分钟结算 + 写入 DB
│                · 滑动窗口检测 + 通知
│                · #[tauri::command] 暴露给前端
│                · 系统托盘
└── db.rs      -- rusqlite 读写封装 + 单元测试
```

> 与 PLAN.md 的差异：原计划拆分为 `input/`、`engine/`、`notify.rs`、`commands.rs` 等模块，实际为了快速落地全部集中在 `lib.rs`。后续如需扩展可再拆分。

### 前端（Vue 3）

```
src/
├── i18n/
│   ├── index.ts         -- vue-i18n 配置（zh-CN / en-US）
│   └── locales/
│       ├── zh-CN.ts     -- 中文翻译
│       └── en-US.ts     -- 英文翻译
├── views/
│   ├── Dashboard.vue    -- 今日统计四卡片 + 今日活动（概览/详细切换）
│   └── Settings.vue     -- 提醒偏好滑块（自动保存）+ 启动行为开关 + 语言切换 + 更新/链接
├── components/
│   ├── Timeline.vue         -- 24h × 60min 色块热力图（CSS Grid）
│   └── TimelineWindows.vue  -- 概览 block 卡片网格（自适应列数，点击展开整行）
├── utils/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lanxiuyun/Catrace](https://github.com/lanxiuyun/Catrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
