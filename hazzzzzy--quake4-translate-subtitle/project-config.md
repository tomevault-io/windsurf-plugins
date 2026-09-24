---
trigger: always_on
description: ﻿# Quake 4 简体中文汉化工程说明
---

﻿# Quake 4 简体中文汉化工程说明

## 项目定位

本工程为 Quake 4（2005，Raven Software）的简体中文汉化与字幕开发环境。运行时使用开源引擎 idTech4A++ 加载玩家合法持有的 Quake 4 1.4.2 游戏数据，不修改原版游戏目录，并保留官方 `q4game.dll`备份。

主要功能包括：

- UI、菜单、任务文本和剧情对白的简体中文翻译。
- 自研语音字幕系统，覆盖普通对白、无线电、PA 广播和补齐的 AI 语音。
- 基于距离、PVS 和友军容差的字幕可听性门控。
- 自研 CJK 字体导出，包含 2 倍超采样、16:9 横向预压缩、垂直对中和材质别名压缩。
- Strogg 外星文字体，以及改造后转译为可读中文的动画与字体切换。
- 基于 pak021 GUI 底稿的 HUD 和交互面板汉化。
- 规划中的独立英文字幕模式只增加字幕，保留原版英文界面与字体。

## 运行拓扑

| 用途 | 路径 | 说明 |
|---|---|---|
| 原版游戏数据 | `D:\Quake 4\q4base` | `fs_basepath`，必须包含 `pak001.pk4`和 1.4.2 的 `pak021.pk4` |
| 工程引擎 | `idTech4Apx\quake4` | idTech4A++ 的 `Quake4.exe`及当前编译的 `q4game.dll` |
| 工程运行数据 | `savedata\q4base` | `fs_savepath`，部署字体、字符串、GUI、lipsync、语音别名、配置、日志和存档 |
| 已知可玩基线 | `D:\Quake4-CN` | 外部对照目录；未经用户明确许可不要覆盖或同步修改 |
| 测试启动 | exe 安装版 `Quake4中文启动器.exe`（Q4CNLauncher） | 安装后用快捷方式启动；安装器安装时从原版 Quake4.exe 提取图标。便携 ZIP 已停发 |

测试统一用 exe 安装版（安装到游戏目录后点 Quake4中文启动器.exe），不再用工程 .cmd 启动器。引擎中文、宽字符 GUI、高清字体、字幕、阴影等选项由启动器/安装器配置。

## 目录职责

### 工程根

工程根就是公开 Git 仓库，远端目标名称为 `Quake4-Translate-Subtitle`。主要保存可开源内容：

- `src/translations`：翻译主表，是中文字符串的源数据。
- `src/tools`：lang、字体、HUD、主菜单、腕表、无线电 decl 和分发资产生成工具。
- `src/engine-patches/Subtitles.h/.cpp`：字幕系统源码。
- `src/engine-patches/0001-quake4-cn-runtime.patch`：基于固定上游标签的完整 8 文件引擎补丁。
- `dist`：公开可分发内容，不等同于本工程当前部署快照。
- `.github/workflows/package.yml`：push 验证与打包、tag Release 发布流程。

修改前后都要在工程根执行 `git status --short`，不要覆盖用户或前序会话的未提交改动。

### `diii4a`

idTech4A++ 上游源码，基线为 `v1.1.0harmattan70`。当前 Quake 4 汉化改动位于：

- `Q3E/src/main/jni/doom3/neo/quake4`：字幕挂钩、玩家保存提示修复和 `Subtitles.cpp/.h`。
- `Q3E/src/main/jni/doom3/neo/CMakeLists.txt`：将字幕实现加入 q4game 构建。

该目录是独立的上游 Git 工作副本，不纳入根仓库。当前权威补丁是 `src/engine-patches/0001-quake4-cn-runtime.patch`，包含 8 个受跟踪文件，并另外复制 `Subtitles.h/.cpp`。`tmp/scripts/quake4-cn-engine-full.patch`只保留为本机交接副本。

### `assets`

保存不能从公开仓库直接生成或需要固定底稿的开发输入：

- `SourceHanSansSC-Medium.otf`：CJK 字体源。
- `hud_pak021_stock.gui`：Quake 4 1.4.2 HUD 底稿。
- `english_fonts`：原版英文字体缓存。

### `savedata/q4base`

当前工程运行时部署目录。主要内容：

- `fonts/chinese`：当前启用的中文字体和 fontdat。
- `strings`：5 个中文 lang 文件。
- `guis`：HUD、字幕、主菜单、腕表、Strogg 面板和转译动画。
- `lipsync`：无线电及 AI 补齐 decl。
- `materials`：字体材质别名。
- `zzz_vo_chinese_alias.pk4`：中文语音路径别名。
- `savegames`、`screenshots`、`qconsole.log`：运行时数据，不是翻译源文件。

`fonts/chinese_bak_r4`及 `poc_*.cfg`为历史备份或测试资产，不应进入正式分发包。

### `tmp`

构建、验证和临时产物目录：

- `scripts/quake4-cn-engine-full.patch`：完整引擎补丁的本机交接副本；公开权威文件位于 `src/engine-patches`。
- `build-q4-ninja-only`：当前 q4game 增量构建目录。
- `windows-sdk-nuget`：本地 Windows SDK 10.0.26100，不是系统级安装。
- 其余截图、隔离存档和生成目录均为测试证据；删除前必须征得用户确认。

### `docs`

按迭代记录的技术沉淀。先读 `docs/MEMORY.md`索引，再按问题读取对应文档。关键主题：

- `quake4-feedback-fixes-r4.md`：GUI 存档结构、pak021 底稿和旧存档状态覆盖。
- `quake4-r6-crash-and-subtitles.md`：换图崩溃、字幕 GUI 指针生命周期及后续版本记录。
- `quake4-font-aspect-and-size.md`：字体比例与体积优化。
- `quake4-strogg-changeover.md`：Strogg 转译动画与字体语义。
- `quake4-dist-package.md`：分发包结构与打包要求。

## GUI 与存档的绝对约束

Quake 4 会按 GUI 源文件结构和窗口顺序序列化状态。必须遵守：

1. HUD 覆盖必须以 `assets/hud_pak021_stock.gui`为底稿。
2. 为兼容存档，原则上只改既有窗口的数值属性；禁止随意增删 `windowDef`、脚本、变量或改变顺序。
3. 旧存档会恢复保存时的 `rect`、`textscale`、文本和其他 GUI 状态，从而暂时覆盖磁盘上的新配置。
4. 视觉验收应从主菜单进入新流程、换图，或在明确的调试场景执行 `reloadGuis all`；不能用旧 `gamestart`判断当前 HUD 文件是否正确。

## GUI 加载机制与字体共用约束

### 两类 GUI 的加载路径（2026-08-01 摸清）

- **走 savepath 的 GUI**（loose 覆盖生效）：HUD（hud.gui/hud_strogg.gui）、主菜单、腕表、字幕、Strogg 面板、电梯/撤离/生命补给等。由 `build_dist_extras.py` 从原版 pak 现场生成到 `savedata/q4base/guis/`，loose 文件覆盖 pak 原版。
- **不走 savepath 的 GUI**（只认 basepath pak）：武器 viewmodel 弹药 GUI（`machinegun_ammo`/`hyperblaster_ammo`/`shotgun_ammo`/`nailgun_ammo`/`rocketlauncher_ammo.gui`）。由武器 def 从 basepath pak 直接加载，不搜 `fs_savepath`，改 rect/font/textscale 不生效。
- **cursor.gui 走 savepath 但禁止修改**（2026-08-03 摸清）：cursor.gui 和 HUD 走同一个 `FindGui → InitFromFile → fileSystem->ReadFile` 路径，loose 文件能被加载。但 cursor GUI 被 `Player.cpp` 的 `WriteUserInterface` 序列化到存档，**修改 cursor.gui 内容会导致存档加载崩溃**（实证：matcolor_x→matcolor_r 等长替换、matcolor 整体替换、3 行合 1 行，全部崩溃；原版不修改不崩溃）。之前"改 cursor 属性纹丝不动"的实测实际是修改后崩溃或存档覆盖所致。**修复准心相关问题只能改引擎 `ui/Window.cpp`（需编译 Quake4.exe），不能改 cursor.gui**。

### marine 字体被三处共用，字形无法分别

marine 基础段（ASCII/英文/数字）被以下三处共用，字形必须一致：

1. **枪身弹药数字**（武器 viewmodel ammo GUI，`font "fonts/marine"`）
2. **准心人名**（`cursor.gui` 的 `crossName` 控件 `gui::npc`，`font "fonts/marine"`）
3. **MCC 终端/医疗面板/简报/监控的数字编号**（各终端 GUI，`font "fonts/marine"`）

marine 恢复原版基础段后，这三处的英文/数字**同时变原版方正字形**，无法只让枪身是原版、准心是思源。要让准心英文名回思源只能 marine 整体回思源（枪身数字也回思源）。当前（2026-08-01）选择 marine 原版基础段：三者统一原版，并附数字位置补偿。

### 改 viewmodel/准心数字位置：改 fontdat，不要改 GUI rect

武器弹药数字和准心的位置由 fontdat 里数字字形的 `top`（垂直）和 `xSkip`（水平 center 基准）决定，`PaintChar` 直接读取这两个度量。GUI rect 改动对这些 viewmodel/准心 GUI 无效（不走 savepath），**改 fontdat 才有效**（2026-08-01 大补偿实证：fontdat 数字 top-10/xSkip-10 立刻右下移）。marine 数字位置补偿（`top-4`/`xSkip-15`，实机标定：垂直下移到不偏上、水平 center 右移到居中）集成在 `export_font.py` 的 `use_original_base` 分支（marine 基础段拷贝后改数字 0-9 的 top/xSkip 再写）；`patch_marine_numoffset.py` 是实机快速微调工具（每次基于 pak 原版重新算，不累积偏移）。注意 xSkip 大幅缩小会让多字符（如机枪"24"）字间距变窄，单字符（霰弹枪）不受影响。

## 引擎构建与部署

### 编译目标分离（2026-08-03 摸清）

idTech4A++ 的 CMake 有两个独立构建目标，源码归属不同：

| 目标 | CMake 选项 | 包含源码 | 产物 |
|---|---|---|---|
| q4game.dll | `QUAKE4=ON` | `quake4/*.cpp`（游戏逻辑、AI、武器、字幕等） | q4game.dll |
| Quake4.exe | `RAVEN=ON` | `ui/*.cpp`（Window/GuiScript/Winvar 等）+ 渲染器 + 框架 + 声音（`src_core_raven`） | Quake4.exe |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hazzzzzy/Quake4-Translate-Subtitle](https://github.com/hazzzzzy/Quake4-Translate-Subtitle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
