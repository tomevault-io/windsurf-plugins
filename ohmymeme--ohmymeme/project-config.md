---
trigger: always_on
description: 轻量化跨平台表情包管理系统，突破表情包数量限制，支持全局快捷键呼出、搜索复制、FTP/S3/R2 同步。
---

# OhMyMeme — AI Agent Guide

## 项目概述
轻量化跨平台表情包管理系统，突破表情包数量限制，支持全局快捷键呼出、搜索复制、FTP/S3/R2 同步。

## 架构
```
系统托盘 (pystray) ↔ 全局快捷键 (keyboard/pynput/轮询)
        ↓ show/hide
WebView 窗口 (pywebview) → Bottle HTTP 服务器 (localhost)
        ↓ JS API 桥 (pywebview.api.method)
JsApi / SettingsApi → SQLite (WAL) + 本地缓存 + 远端同步
```

## 技术栈
- **Python 3.12** + **pywebview** (frameless 窗口) + **Bottle** (静态文件/缩略图路由)
- **SQLite** (WAL, `threading.local()` 连接, `threading.Lock()` 写锁)
- **PIL/Pillow** (缩略图, 剪贴板图像)
- **pystray** (托盘, 惰性导入避免 headless CI 崩溃)
- **InnoSetup** (Windows 安装包) / **PyInstaller** (打包)
- **GitHub Actions** (lint+test on Ubuntu, build+installer on Windows)

## 核心原则
- **不得重构该项目** — 仅做最小必要修改，不改变现有架构、设计模式、代码组织
- **尽量不创建新文件** — 优先修改现有文件
- **增改同步** — 增加新功能或创建新文件后，同步修改 `README.md` 和 `AGENTS.md` 中对应描述
- **关联文件同步** — 修改后检查是否需要同步更新 `.gitignore`、`Makefile`、`pyproject.toml`、`requirements.txt`、`environment.yml` 等关联文件

## 代码规范
- 无类型标注（`database.py`/`updater.py` 除外可使用 `typing` 基本类型）
- 无非必要注释（除非用户明确要求）
- 每段函数需要有简单功能注释
- 无 emoji（除非用户要求）
- 无文档字符串（只对公开 API 使用极简单行 docstring）
- 无冗余前缀/后缀说明（写完代码即结束，不加总结）

## 格式 & Lint
- `black src/` (line-length 88,  black 26.5.1)
- `ruff check src/` (select F, E, W, I)
- 新增依赖同时更新 `requirements.txt` 和 `environment.yml`
- **PR 贡献必须确保 `black --check src/` 和 `ruff check src/` 全部通过**，CI 会检查这两项

## 关键目录
```
src/              # 主代码
  main.py         # CLI 入口, OhMyMemeApp 编排
  webui.py        # pywebview 窗口 + JsApi/SettingsApi + Bottle 路由
  updater.py      # 版本检查 + 并发镜像下载
  database.py     # MemeDB (SQLite, 6 表)
  config.py       # Config (JSON + Fernet 加密密钥)
  sync.py         # 同步后端 (FTP/S3/R2)
  tray.py         # TrayManager (pystray, 惰性导入)
  hotkey.py       # GlobalHotkey (三级降级: keyboard→pynput→轮询)
  clipboard_util.py # 剪贴板操作 (Win32 ctypes / macOS osascript / Linux xclip)
  gif_stego.py     # GIF 增量隐写（实验性：粘贴表情大小 + 无损还原原图）
  crypto_util.py  # 加密 (Fernet + PBKDF2, 降级 XOR)
  manifest.py     # meme-index.json 构建/加载
  platform_util.py # 平台工具 (WSL检测, 开机自启)
  adb_util.py      # ADB 自动检测/下载 + QQ 表情包缓存导入（ADB 拉取 + 魔数识别扩展名 + ZIP 打包）
  qqnt_extract.py  # QQNT 本地收藏表情提取（GPL-3.0 衍生模块，纯函数 + 回调接口，无 UI 依赖）
  webui/          # 前端静态文件
    index.html    # 主窗口 HTML+CSS+JS
    settings.html # 设置窗口 HTML+CSS+JS
scripts/
  build.py        # PyInstaller + InnoSetup 构建脚本 (i18n zh/en)
  launcher.py     # PyInstaller 入口
tests/
  test_core.py    # unittest 风格: Version/Config/Crypto/Database
  test_startup.py # pytest 风格: 全生命周期集成测试
```

## js_api 桥接规范
- `JsApi` 暴露给主窗口，`SettingsApi` 暴露给设置窗口
- JS 调用: `pywebview.api.methodName(...args)` → 自动序列化
- JS 辅助函数: `async function api(method, ...args) { return await pywebview.api[method](...args); }`
- 返回类型: `str` / `int` / `bool` / `dict` / `list`，错误返回 `None` 或 `{"ok": false, "error": "..."}`
- 图片传输: 缩略图通过 `/api/thumb/{id}` HTTP 路径渲染，不通过 JS API JSON

## 关键实现细节

### 系统托盘
- `TrayManager` 在 daemon 线程运行
- 惰性导入: `_pystray_ok()` 避免 headless CI (X11 `DisplayNameError`)
- WSL 自动跳过托盘

### 全局快捷键
- 三级降级: `keyboard` → `pynput` → 200ms 轮询 (`keyboard.is_pressed`)
- WSL 无法捕获全局快捷键

### 窗口
- 主窗口 ~700×500 frameless, 设置窗口 460×560 frameless
- 自定义 JS 拖拽: 鼠标事件 → `pywebview.api.move_window(dx, dy)`
- 增量回退（Windows/macOS）用 `screenX/screenY`（**勿改 `clientX/clientY`** — clientX 是相对窗口坐标，窗口自身滞后位移会被下一次 mousemove 当作反向增量回传，形成反馈振荡导致高频抖动）；Linux 走合成器原生拖动不经过此路径
- **Linux 拖拽必须走合成器**：`w.move()` 在 Wayland 下无效（合成器不允许客户端自定位），mousedown 时 JS 调 `start_window_drag()` → 后端 `GLib.idle_add(native.begin_move_drag, ...)` 交给合成器交互式拖动；时间戳用 `Gdk.CURRENT_TIME`（GDK 文档允许未知时间时用它，X11 回填最近输入事件时间、Wayland 不参与）
- `#titlebar` 上可拖拽 (排除 `.title-btn` 按钮区域)

### 数据库
- 7 表: `memes`, `tags`, `meme_tags`, `collections`, `meme_collections`, `favorites`, `recent_uses`
- `PRAGMA journal_mode=WAL`, `PRAGMA foreign_keys=ON`
- `MemeDB.search()`: 动态 WHERE, 多标签交集用 `HAVING COUNT = len(tags)`
- `memes.sort_order`: 自定义排序（拖拽更新），默认 0，查询 `ORDER BY sort_order ASC, updated_at DESC`
- `collections.parent_id`: 多级分组支持（最多 3 层），`NULL` 为顶层分组
- `meme_collections.sort_order`: 分组内成员自定义排序
- `recent_uses`: `meme_id` + `used_at`，复制时 `INSERT OR REPLACE`，按 `used_at DESC` 取最近使用

### 配置
- `%APPDATA%/OhMyMeme/config.json` (Win), JSON 格式
- 密钥字段 (ftp_password, s3_secret_key 等) 用 Fernet 加密存储
- 全局单例: `get_config()`, `get_db()`

### 同步
- manifest 文件: `meme-index.json`
- SHA-256 差异对比, `push(delete_remote)`/`pull(remove_local)`
- 远端路径: `{root}/memes/`, `{root}/meme-index.json`
- 同步进度: `_sync_state` 全局变量追踪进度，`get_sync_progress()` 供 JS 轮询
- `push()`/`pull()` 内循环中更新 files_done/bytes_done/current_file 等字段
- 前端 300ms 轮询 `get_sync_progress()` 显示进度条 + 实时速度
- 设置页 4 个开关控制进度/完成弹窗是否显示
- **多线程传输**: `push()`/`pull()` 使用 `ThreadPoolExecutor`，每个线程创建独立后端连接
- 并发数: 配置项 `sync_threads`（默认 3，范围 1-8），通过 `config.json` 或 `SettingsApi` 修改
- `_push_worker`/`_pull_worker`: 接收文件子列表，操作独立后端连接，原子递增 `_sync_state`
- `_sync_lock` (`threading.Lock`) 保护 `_sync_state` 写操作；`_increment_sync_progress()` 提供原子递增
- `_chunk_list(lst, n)` 将文件列表均匀切分给各线程

### 更新
- GitHub API 查询: `/releases/latest` → `/releases?per_page=5` 回退
- 镜像并发: `_urlopen_mirror` / `_urlretrieve_mirror` 用 `ThreadPoolExecutor` + `as_completed`
- 镜像列表: `github.dpik.top` → `gh.dpik.top` → `gh-proxy.org` → 自建镜像（仅用于版本查询）→ 直连 GitHub
- 下载进度: `start_download()` → 后台线程 → JS 每 500ms 轮询 `get_download_progress()`

### 环境检测
- WSL 检测: `/proc/version` 包含 "microsoft"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OhMyMeme/OhMyMeme](https://github.com/OhMyMeme/OhMyMeme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
