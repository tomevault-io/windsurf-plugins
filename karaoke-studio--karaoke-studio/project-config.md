---
trigger: always_on
description: 给 Claude Code / Codex / 其他 coding agent 看的项目入口。读完应该能在不打扰用户的前提下定位代码、跑测试、走完发版流程。
---

# AGENTS.md

给 Claude Code / Codex / 其他 coding agent 看的项目入口。读完应该能在不打扰用户的前提下定位代码、跑测试、走完发版流程。

---

## 0. 用户持久化偏好（最高优先级）

以下规则适用于本仓库的所有新会话，除非用户在当前请求中主动、明确地要求例外：

1. **禁止创建新分支**：始终在当前分支（通常为 `main`）工作、提交和推送。不得因 agent、skill、插件或通用 Git 工作流的默认建议自行创建功能分支；只有用户明确要求“创建分支”时才允许创建。
2. **禁止新建文档**：不得自行新增计划、说明、报告、设计稿、Markdown 或其他文档文件。只有用户明确要求“新建文档”时才允许创建；现有文档仅在任务确有必要时原地更新。

---

## 1. 这是什么

**Karaoke Studio（卡拉OK工作台）**——面向卡拉 OK / B 站投稿制作的 Windows 桌面工具，Python + PyQt6 + PyInstaller，仓库主体是 `karaoke-studio/karaoke-studio`，由 [Myosotis11037](https://github.com/Myosotis11037)（原 karaoke-helper 作者）与 [Xuan-cc](https://github.com/Xuan-cc)（原 StrangeUtaGame 作者）于 2026-06 合并而成。详情见 [AUTHORS.md](AUTHORS.md) / [NOTICE](NOTICE)。

UI 的核心是 [`WORKFLOW_STEPS`](krok_helper/gui_qt.py)（约第 1064 行）定义的 **6 步工作流**：

| 步骤 | 模块 | 实现位置 |
|---|---|---|
| 1 | 视频下载 | [`krok_helper/video_download/`](krok_helper/video_download/) |
| 2 | 波形对齐 | [`krok_helper/audio_alignment.py`](krok_helper/audio_alignment.py)（999 行） |
| 3 | 歌词检索 | [`krok_helper/lyrics.py`](krok_helper/lyrics.py)（1426 行） |
| 4 | 歌词打轴 | [`krok_helper/lyrics_timing/`](krok_helper/lyrics_timing/) — **SUG submodule** |
| 5 | 字幕视频生成 | [`krok_helper/subtitle_render/`](krok_helper/subtitle_render/)（已合入 `main`；详见 §9） |
| 6 | Hi-Res 混流 | [`krok_helper/pipeline.py`](krok_helper/pipeline.py) |

---

## 2. 仓库结构

```
karaoke-studio/
├── app.py                          # 入口 → krok_helper.cli.main
├── krok_helper/
│   ├── __init__.py                 # 把 lyrics_timing/src 加到 sys.path
│   ├── cli.py                      # argparse + 启动 GUI
│   ├── config.py                   # APP_VERSION 等常量
│   ├── gui_qt.py                   # 主窗口、7000+ 行，所有 UI 都在这里
│   ├── pipeline.py                 # Hi-Res 混流流水线
│   ├── audio_alignment.py          # 波形对齐
│   ├── lyrics.py                   # 歌词检索 + 转换
│   ├── settings.py                 # 用户设置序列化
│   ├── ffmpeg.py / network.py / windows.py
│   ├── updater/                    # 自动更新客户端（worker / installer / sources / settings）
│   ├── updater_app/                # 独立 Updater.exe（PyInstaller 单独打包）
│   ├── video_download/             # yt-dlp 封装
│   ├── lyrics_timing/              # ⚠️ Git submodule（StrangeUtaGame）— 不要直接改源码
│   └── assets/                     # 图标、logo、平台 SVG
├── scripts/
│   ├── build_windows.bat           # 本地 + CI Windows 打包
│   └── build_macos.command         # 本地 + CI macOS 打包
├── tests/                          # pytest，对应主程序各模块
├── docs/
│   ├── release-process.md          # 发版流程（committed）
│   └── release-runbook.local.md    # 个人 cheat sheet（gitignored）
├── .github/workflows/release.yml   # tag v* 触发 → 打包 → 发布 GitHub Release
├── CHANGELOG.md / README.md / AUTHORS.md / LICENSE / NOTICE
└── AGENTS.md                       # 本文件
```

---

## 3. 跑起来

```powershell
# 带 PyQt6 的 Python 解释器
C:\Python314\python.exe app.py
```

CLI 选项见 [`krok_helper/cli.py`](krok_helper/cli.py)（`--video` / `--on-audio` / `--off-audio` / `--output-dir` / `--ffmpeg-dir` 等）。无参数则直接进 GUI。

测试：

```powershell
C:\Python314\python.exe -m pytest tests\
```

Qt 嵌入冒烟（无显示器环境）：

```powershell
$env:QT_QPA_PLATFORM='offscreen'
C:\Python314\python.exe -c "from PyQt6.QtWidgets import QApplication; app=QApplication([]); from krok_helper.gui_qt import KrokHelperQtApp; w=KrokHelperQtApp(); print(type(w.lyrics_timing_page).__name__)"
```

---

## 4. Submodule（StrangeUtaGame）边界

`krok_helper/lyrics_timing/` 是独立仓库 [`karaoke-studio/StrangeUtaGame`](https://github.com/karaoke-studio/StrangeUtaGame) 的 submodule。

**规则**：

- **不要直接改 `krok_helper/lyrics_timing/src/strange_uta_game/` 里的代码**。要改先去 SUG 仓库提 PR，merge 后再 bump submodule。
- 嵌入主程序的入口是 [`krok_helper/gui_qt.py`](krok_helper/gui_qt.py) 里的 `self.lyrics_timing_page`（约第 2380 行附近构造）。SUG 自己暴露 `MainWindow` class，宿主把它当一个 Qt widget 挂上去。
- 嵌入契约见 SUG 仓库的 `docs/embedding-contract*.md`（在 submodule 内）。
- [`krok_helper/__init__.py`](krok_helper/__init__.py) 会自动把 `lyrics_timing/src/` 加进 `sys.path`，所以 `import strange_uta_game` 在主程序里直接可用。

**新会话第一件事**：

```powershell
git submodule status
```

如果显示 `-<sha>`（前面有 `-`），说明 submodule 没初始化，跑 `git submodule update --init --recursive` 否则什么都跑不起来。

---

## 5. 自动更新机制

- [`krok_helper/updater/worker.py`](krok_helper/updater/worker.py)：在主程序里跑，查询 GitHub Releases API（全 403 时用 github.com 网页 302 跳转兜底），对比 `APP_VERSION`，跨版本聚合 changelog。
- [`krok_helper/updater_app/`](krok_helper/updater_app/)：独立 GUI `Updater.exe`（复用 SUG `updater_app` 的 PyQt6 界面与增量更新逻辑），主程序退出后由它显示进度、替换文件并重启，不弹控制台。需要 `build_updater.py` 单独打包。
- **增量更新**：[`scripts/build_parts.py`](scripts/build_parts.py) 产出 `KaraokeStudio-windows.json`（manifest）+ `-app.zip` + `-runtime.zip`；Updater 按 manifest diff 只下变化的 part，失败自动回退全量 zip。依赖未变时 CI 复用上一版 runtime zip 原文件（`--require-runtime-reuse` 安全闸）。完整机制、配置与失败矩阵见 [`docs/auto_update.md`](docs/auto_update.md)；设计取舍见 [`docs/工作台更新器完善计划.md`](docs/工作台更新器完善计划.md)。
- 资产命名是硬编码的：`KaraokeStudio-windows.zip` / `KaraokeStudio-macos.zip`（见 `worker.current_asset_name()`）。**manifest 名 `KaraokeStudio-windows.json` 由存量客户端从 zip 名派生，全都不可改**。改名要改四处：worker、`scripts/build_*`、`scripts/build_parts.py`、workflow。
- **更新弹窗会直接展示 GitHub Release 的 body**，所以 release body 必须是中文。详见 §6。

---

## 6. 发版

所有发版规则在 [`docs/release-process.md`](docs/release-process.md)，**必读**。要点：

- 工作台与 submodule **不会同时发版**，分两条流程：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karaoke-studio/karaoke-studio](https://github.com/karaoke-studio/karaoke-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
