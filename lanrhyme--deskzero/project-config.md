---
trigger: always_on
description: Tauri v2 桌面应用（仅 Windows）。React 19 + TypeScript 前端，Rust 后端。窗口嵌入 Windows 桌面图标层（壁纸与图标之间）。
---

# DeskZero

Tauri v2 桌面应用（仅 Windows）。React 19 + TypeScript 前端，Rust 后端。窗口嵌入 Windows 桌面图标层（壁纸与图标之间）。

## 命令

```bash
npm run dev          # Vite 开发服务器（端口 1420）
npm run build        # tsc -b && vite build
npm run tauri dev    # Tauri 开发模式（自动启动前端）
npm run tauri build  # 构建发布二进制
```

前端无 lint、typecheck、test、formatter 脚本。用 `tsc -b`（no emit）做类型检查。

Rust 测试：在 `src-tauri/` 下执行 `cargo test`。

## 架构

### 前端 (`src/`)

- 入口：`main.tsx` -> `App.tsx`，按 URL 路径路由（`/` = 桌面，`/settings` = 设置页）
- 状态管理：Zustand stores（`stores/`）：`containerStore`、`desktopStore`、`settingsStore`
- 服务层（`services/`）：通过 Tauri `invoke()` 与 Rust 后端 IPC 通信
- 路径别名：`@/*` 映射到 `./src/*`
- Tailwind CSS v4（CSS 配置，`styles/globals.css` 中 `@import "tailwindcss"`）
- 主题：CSS 变量在 `:root` 和 `[data-theme="dark"]`，通过 `data-theme` 属性切换
- 工具函数：`cn.ts` 使用 `clsx` + `tailwind-merge`

### Rust 后端 (`src-tauri/src/`)

- 入口：`main.rs` -> `lib.rs::run()` — 初始化 Tauri，嵌入窗口到 Windows 桌面层
- `commands/` — Tauri invoke 处理器：`container`、`desktop`、`file`、`system`、`backup`
- `models/` — 数据类型：`container`、`item`、`settings`、`backup`
- `storage/` — SQLite 持久化（`rusqlite` bundled）：`db.rs`（初始化/建表/PRAGMA 配置）、`container_store`、`settings_store`、`desktop_store`、`backup_store`、`migration`
- `backup_timer.rs` — 后台自动备份定时器（tokio 异步循环）
- `desktop/` — Windows 桌面集成：`icon_scanner`、`shortcut`、`watcher`
- `clipboard.rs` — 文件剪贴板操作
- `context_menu.rs` — Windows 右键菜单

### 关键行为

- 主窗口启动时隐藏，嵌入 Windows Progman/WorkerW 层（最多重试 3 次）
- 设置窗口是独立 Tauri 窗口，通过事件（`settings-updated`）与主窗口通信
- 桌面文件监视器通知前端文件系统变化
- Release 配置：`lto = true`、`opt-level = "s"`、`strip = true`
- 数据库路径：`%APPDATA%/DeskZero/deskzero.db`
- 存储初始化流程：`init_db()` 建表 -> `run_migrations()` 迁移
- 备份系统：`backup_store` 管理快照 CRUD，`backup_timer` 后台定时自动备份，`BACKUP_LOCK` 互斥锁保护所有存储操作

## 持久化规范

本项目全面采用 SQLite（通过 `rusqlite` + `bundled` 特性）作为本地唯一存储引擎，彻底摒弃手动读写 JSON 文件的做法。此规范适用于任何未来负责本项目的 AI 或开发者。

### 核心原则

1. **零文件操作**：绝对禁止直接使用 `fs::write` 或 `fs::read_to_string` 来保存/读取包含核心业务数据的配置文件。所有的持久化数据（布局、容器、设置、甚至后续的主题缓存等）必须进入 `deskzero.db`。
2. **ACID 事务安全**：针对批量插入或更新（如：保存容器及其内部的项目），必须开启数据库事务 `tx = conn.transaction()` 以保证原子性，防止一半写入成功一半失败。
3. **单向信任**：前端通过 Tauri 的 `invoke` 调用修改数据时，应该在后端验证数据的有效性然后使用带参数绑定的 SQL 语句插入数据库，绝不允许拼接 SQL（防注入）。

### 后端模块结构

- **`src/storage/db.rs`**：负责数据库初始化（`init_db`），包含所有 `CREATE TABLE IF NOT EXISTS` 语句。每次添加新表都必须在此注册。`get_connection()` 中必须执行 `PRAGMA foreign_keys = ON` 以确保外键约束生效。
- **`src/storage/migration.rs`**：用于处理老旧格式的迁移。
- **特定领域的 Store 文件**（如 `settings_store.rs`, `container_store.rs`）：存放领域模型的数据库 CRUD 方法，它们只应暴露出对模型的操作，如 `load_containers()` 和 `save_containers()`，而封装掉所有 SQL 细节。

### 添加新持久化特性的标准流程

假设你要添加一个“小部件（Widget）”特性：

1. **定义模型**
   在 `models/` 目录下定义你的 Rust `struct`，并实现 `Serialize` / `Deserialize`。
2. **定义表结构**
   在 `storage/db.rs` 的 `init_db` 中加入建表语句，如 `CREATE TABLE IF NOT EXISTS widgets (id TEXT PRIMARY KEY, type TEXT, x REAL, y REAL, config TEXT)`。如果属性灵活多变，可以使用 `config TEXT` 字段存放序列化的 JSON。
3. **实现 Store**
   创建 `storage/widget_store.rs`，编写 `load_widgets` 和 `save_widgets`。
   * 读取时使用 `stmt.query_map` 将 Row 映射为模型。
   * 写入时使用差异删除 + `INSERT ... ON CONFLICT(id) DO UPDATE SET ...`（UPSERT），禁止先全量 `DELETE` 再 `INSERT`（参见健壮性规范）。
4. **导出接口**
   如果需要在前端访问，通过 Tauri `#[tauri::command]` 暴露出去。

### 数据模型映射约定

- **基础数据类型**：对应 SQL 的 `TEXT`, `REAL`, `INTEGER`。
- **枚举 (Enum)**：序列化为字符串存入 `TEXT` 字段，读取时通过 `serde_json::from_str(&format!("\"{}\"", val))` 解析恢复。枚举必须包含 `Other(String)` 变体（参见健壮性规范）。
- **灵活的深层对象配置 (Styles/Config)**：对非核心检索字段，例如复杂的样式配置，允许将其序列化为 JSON 字符串存入 `TEXT` 字段，以规避建立过多关联表和提高扩展性。JSON 序列化的结构体必须包含 `#[serde(flatten)] extra: HashMap<String, serde_json::Value>` 字段（参见健壮性规范）。

## 约定

- 无 ESLint、Prettier、CI 配置
- Rust 和 TypeScript 代码注释使用中文，保持此风格
- Rust 测试：单独的 `*_test.rs` 文件，通过父模块 `mod.rs` 中 `#[cfg(test)] mod xxx_test;` 引入
- 前端无测试
- Rust 调试日志用 `eprintln!`（终端可见，应用内不显示）
- 提交信息应使用中文

## 可复用 UI 组件

所有设置面板和配置 UI 必须使用 `src/components/UI/` 下的共享组件，禁止内联重复实现。

### 组件清单

| 组件 | 文件 | 用途 |
|------|------|------|
| `SwitchToggle` | `UI/SwitchToggle.tsx` | 开关切换，封装 `@headlessui/react` Switch |
| `Slider` | `UI/Slider.tsx` | 滑块，自定义 pointer-event 实现 |
| `NumberInput` | `UI/NumberInput.tsx` | 数字步进器（+/- 按钮 + 文本输入） |
| `ColorPicker` | `UI/ColorPicker.tsx` | Canvas 2D 色板 + hex 输入 + 预设色块 |
| `SegmentedControl` | `UI/SegmentedControl.tsx` | 分段按钮组（如 主题切换 light/dark/system） |
| `CustomSelect` | `UI/CustomSelect.tsx` | 自定义下拉选择器，支持 top/bottom 定位 |
| `TextInput` / `TextArea` | `UI/TextInput.tsx` | 文本输入框 / 多行文本域 |
| `SettingRow` | `UI/SettingRow.tsx` | 设置行布局（标题 + 描述 + 控件），支持 horizontal/vertical |
| `ConfirmDialog` | `UI/ConfirmDialog.tsx` | 确认对话框，framer-motion 动画 |
| `ToastContainer` | `UI/ToastContainer.tsx` | Toast 通知 |

### 使用规范

1. **禁止内联重复实现**：不得在设置面板中自行实现 toggle、slider、color picker 等控件，必须引用共享组件。
2. **设置面板 portal 模式**：所有容器/小组件的设置面板必须通过 `createPortal` 渲染到 `document.body`，外层包裹 `<div className="fixed inset-0 z-[99] settings-backdrop">` 阻止事件穿透。
3. **事件隔离**：设置面板的 backdrop 必须阻止 `onPointerDown` 和 `onClick` 冒泡，桌面层的 `onPointerDown` / `onDoubleClick` / `handleContextMenu` 需检查 `.settings-backdrop` 守卫。
4. **容器 `touch-none`**：可拖拽容器的根 `motion.div` 必须包含 `touch-none` class，否则桌面框选会穿透。
5. **新增共享组件**：当两个及以上面板出现相同 UI 模式时，应提取为 `src/components/UI/` 下的共享组件。

## 代码健壮性规范

此规范确保数据在跨版本升级、功能增减时保持安全，不丢失、不损坏。适用于任何未来负责本项目的 AI 或开发者。

### 1. 前后兼容的序列化

#### 枚举必须包含 `Other(String)` 变体


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LanRhyme/DeskZero](https://github.com/LanRhyme/DeskZero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
