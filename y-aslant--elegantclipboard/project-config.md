---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

### 开发命令

```bash
# 安装依赖
npm install

# 启动开发模式（前端 + 后端）
npm run tauri dev

# 仅启动前端开发服务器（端口 14200）
npm run dev

# 构建前端
npm run build

# 预览生产构建
npm run preview

# 构建生产版本
npm run tauri build

# 代码检查
npm run lint

# 自动修复代码问题
npm run lint:fix

# Rust 测试
cd src-tauri && cargo test
```

### 最近更新

**v0.x 新功能**：
- **窗口毛玻璃特效**：支持 Mica / Acrylic / Tabbed 三种 Windows 11 DWM 背景特效，Win10 自动回退
- **工具栏自定义**：可配置工具栏按钮的显示、隐藏和排序
- **数据清理**：三级数据清理操作（清空历史 / 恢复默认配置 / 重置所有数据）
- **图片预览窗口**：独立悬浮窗口显示图片，支持缩放和左右定位
- **动态主题系统**：支持 default/emerald/cyan/system 四种主题，system 主题跟随 Windows 系统强调色
- **搜索高亮**：搜索结果自动提取关键词上下文，提升搜索体验
- **一键回到顶部**：滚动超过 200px 显示悬浮按钮
- **窗口状态重置**：关闭时自动重置搜索和滚动位置（可通过 `autoResetState` 配置）
- **文件操作增强**：文件有效性检查、另存为、显示在资源管理器等功能

## 项目架构

ElegantClipboard 是一个基于 Tauri 2.0 的剪贴板管理工具，采用 React 前端 + Rust 后端的架构。

### 整体结构

```
src/                    # React 前端
├── components/
│   ├── ClipboardList.tsx        # 虚拟滚动列表
│   ├── ClipboardItemCard.tsx    # 卡片组件
│   ├── CardContentRenderers.tsx # 内容渲染器（图片/文件预览）
│   ├── settings/                # 设置页面组件
│   └── ui/                      # shadcn/ui 基础组件（Radix 封装）
├── hooks/
│   └── useSortableList.ts       # 拖拽排序 Hook
├── lib/
│   ├── constants.ts             # 常量（工具栏按钮注册表等）
│   ├── theme-applier.ts         # 主题/窗口特效应用器
│   └── utils.ts                 # 工具函数
├── stores/            # Zustand 状态管理
└── main.tsx           # 入口点（简单路由）

src-tauri/              # Rust 后端
├── src/
│   ├── main.rs             # 入口点
│   ├── lib.rs              # 核心库（Tauri 命令注册、窗口管理）
│   ├── config.rs           # 配置文件管理
│   ├── shortcut.rs         # 快捷键解析模块
│   ├── positioning.rs      # 窗口定位（多显示器支持）
│   ├── admin_launch.rs     # 管理员启动功能
│   ├── keyboard_hook.rs    # 窗口状态追踪
│   ├── input_monitor.rs    # 全局鼠标监控（点击外部检测）
│   ├── updater.rs          # 自动更新（GitHub Release 检查/下载，支持系统代理）
│   ├── win_v_registry.rs   # Win+V 替换（注册表）
│   ├── commands/           # Tauri 命令（按功能拆分）
│   │   ├── mod.rs          # AppState 定义 + 模块导出
│   │   ├── clipboard.rs    # 剪贴板 CRUD 命令
│   │   ├── settings.rs     # 设置/监控/自启动命令
│   │   └── file_ops.rs     # 文件操作命令（并行检查）
│   ├── clipboard/          # 剪贴板监控模块
│   ├── database/           # SQLite 数据库
│   └── tray/               # 系统托盘
```

### Tauri 命令架构

**后端（Rust）**：命令按功能模块化组织，在 `lib.rs` 通过 `invoke_handler` 注册。

**命令模块**（`src-tauri/src/commands/`）：
- `clipboard.rs` - 剪贴板 CRUD：`get_clipboard_items`、`get_clipboard_item`、`get_clipboard_count`、`toggle_pin`、`toggle_favorite`、`move_clipboard_item`、`delete_clipboard_item`、`clear_history`、`clear_all_history`、`copy_to_clipboard`、`paste_content`
- `settings.rs` - 设置/监控：`get_setting`、`set_setting`、`get_all_settings`、`pause_monitor`、`resume_monitor`、`get_monitor_status`、`optimize_database`、`vacuum_database`、`reset_settings`、`reset_all_data`、`select_folder_for_settings`、`open_data_folder`、`is_portable_mode`、`is_autostart_enabled`、`enable_autostart`、`disable_autostart`、`get_system_accent_color`
- `file_ops.rs` - 文件操作：`check_files_exist`（rayon 并行）、`show_in_explorer`、`paste_as_path`、`get_file_details`、`save_file_as`

**窗口/系统命令**（`lib.rs`）：
- 窗口管理：`show_window`、`hide_window`、`set_window_pinned`、`set_window_effect`、`open_settings_window`
- 管理员启动：`is_admin_launch_enabled`、`enable_admin_launch`、`is_running_as_admin`
- 快捷键：`update_shortcut`、`enable_winv_replacement`

**前端（TypeScript）**：通过 `@tauri-apps/api/core` 的 `invoke()` 函数调用：

```typescript
import { invoke } from "@tauri-apps/api/core";

const items = await invoke<ClipboardItem[]>("get_clipboard_items", {
    search: query,
    limit: 100
});
```

### 关键架构模式

**1. 仓储模式（Repository Pattern）**
- 位置：`src-tauri/src/database/repository.rs`
- `ClipboardRepository`、`CategoryRepository`、`SettingsRepository`
- 提供数据库 CRUD 操作的抽象层
- 使用 `Arc<Mutex<Connection>>` 实现线程安全

**2. 服务模式（Service Pattern）**
- 位置：`src-tauri/src/clipboard/monitor.rs`
- `ClipboardMonitor` 管理剪贴板监控生命周期
- 使用独立线程运行（`clipboard-master`）
- 通过 Tauri 事件向前端推送更新：`app.emit("clipboard-updated", id)`

**3. 状态管理**
- **前端**：Zustand stores（`src/stores/`）
  - `clipboard.ts` - 剪贴板数据状态
  - `ui-settings.ts` - UI 设置（持久化 + 多窗口同步）
    - `cardMaxLines` - 卡片最大行数
    - `showTime/CharCount/ByteSize` - 元数据显示开关
    - `imagePreviewEnabled` - 图片预览开关
    - `previewZoomStep` - 缩放步进
    - `previewPosition` - 预览位置（auto/left/right）
    - `imageAutoHeight/imageMaxHeight` - 图片高度设置
    - `colorTheme` - 颜色主题（default/emerald/cyan/system）
    - `windowEffect` - 窗口特效（none/mica/acrylic/tabbed）
    - `toolbarButtons` - 工具栏按钮配置
    - `autoResetState` - 关闭时重置状态
- **后端**：`AppState` 通过 Tauri State 共享
  ```rust
  pub struct AppState {
      pub db: Database,
      pub monitor: ClipboardMonitor,
  }
  ```

### 事件驱动通信

**后端 → 前端**：
```rust
// Rust
app_handle.emit("clipboard-updated", id)?;

// TypeScript
import { listen } from "@tauri-apps/api/event";
listen("clipboard-updated", (event) => { ... });
```

**前端 ↔ 前端**（多窗口同步）：
```typescript
import { emit, listen } from "@tauri-apps/api/event";
emit("ui-settings-changed", state);
listen("ui-settings-changed", (event) => { ... });
```

## 窗口配置

主窗口（`main`）采用特殊配置以支持全局快捷键：
- `decorations: false` - 无边框窗口
- `focus: false` - 运行时设置 `set_focusable(false)`
- `alwaysOnTop: true` - 置顶显示

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Y-ASLant/ElegantClipboard](https://github.com/Y-ASLant/ElegantClipboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
