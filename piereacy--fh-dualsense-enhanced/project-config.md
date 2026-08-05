---
trigger: always_on
description: FH-DualSense-Enhanced 读取 Forza Horizon Data Out 的 UDP 遥测，将车辆状态转换为 DualSense 自适应扳机、USB/Bluetooth 握把触觉与可选灯效，并以独立 Windows EXE 或源码方式运行。
---

# Codex 项目工作指引

## 项目目标

FH-DualSense-Enhanced 读取 Forza Horizon Data Out 的 UDP 遥测，将车辆状态转换为 DualSense 自适应扳机、USB/Bluetooth 握把触觉与可选灯效，并以独立 Windows EXE 或源码方式运行。

## 技术栈

- Python `>=3.13`，依赖和运行环境由 `uv` 管理，锁文件为 `src/uv.lock`。
- GUI 使用 `CustomTkinter`，TUI 使用 `Textual`。
- 原生手柄输出使用 `hidapi`；USB 握把触觉使用 `sounddevice`、PortAudio 和 `NumPy`，Bluetooth 握把触觉使用同一 `NumPy` PCM renderer 和 HID report `0x36`。
- Windows Xbox App 输入桥使用项目自有 `ctypes` ABI 封装、固定哈希的 `ViGEmClient.dll` 和可选离线 ViGEmBus 安装器，目标设备为虚拟 Xbox 360 Controller。
- 系统托盘使用 `pystray` 和 `Pillow`，进程检测使用 `psutil`。
- 内置更新器使用 Python 标准库访问 GitHub Releases，并由独立 PyInstaller Helper、持久化事务 journal 和 Windows Shell Link 迁移完成版本化 EXE 更新。
- 测试使用 `pytest`，发布使用 ZUV、锁定为 `6.16.0` 的 PyInstaller 和 GitHub Actions；Windows 构建生成唯一的标准 `FH-DualSense-Enhanced-R<n>.exe`。

## 开始任务前

1. 阅读 `docs/PROJECT_STATE.md`，确认当前阶段、已知问题和未完成验证。
2. 涉及运行链路或模块边界时阅读 `docs/ARCHITECTURE.md`。
3. 涉及既有产品取舍或准备改变行为时阅读 `docs/DECISIONS.md`。
4. 阅读与任务直接相关的代码和测试，不要只依据 README 或旧设计文档。
5. 执行 `git status --short --branch`、`git diff` 和 `git log -10 --oneline`，保留用户已有改动。
6. 用户文档和发行信息以 `README.md`、`LICENSE`、`docs/THIRD_PARTY_NOTICES.md` 和 `.github/workflows/release.yml` 为准，但发现其与代码不一致时必须明确指出。

“老三样”固定指根目录 `AGENTS.md`、`docs/ARCHITECTURE.md` 和 `docs/DECISIONS.md`。行为边界、长期架构或关键产品决策改变时必须检查并同步这三个文件；当前阶段、测试、构建和工作树进度另行同步到 `docs/PROJECT_STATE.md`，不要把临时进度写入老三样。

## 核心入口

| 位置 | 职责 |
| --- | --- |
| `src/main.py`、`src/modules/runtime_logging.py` | CLI、配置加载、GUI/TUI/headless 启动、崩溃日志和有界持久运行日志入口 |
| `src/modules/loop.py` | 遥测热循环、空闲静音、退出检测和输出去重 |
| `src/modules/forzahorizon/udp_listener.py` | UDP 监听、324 字节包解析和原始包转发 |
| `src/modules/forzahorizon/redline.py` | 保留原始 `max_rpm` 的动态断油转速预测、同挡位事件确认和按车辆学习 |
| `src/modules/forzahorizon/game_launch.py` | Windows FH4/FH5/FH6 定义、Steam 安装发现、Xbox AUMID 发现、精确进程检测和显式启动 |
| `src/modules/forzahorizon/fh6_language.py` | FH6 专属语言包内容识别、有效语言摘要、显式交换/还原和崩溃恢复 |
| `src/modules/forzahorizon/controller_icons.py` | FH6 DualSense 图标 MOD 的校验、双目标事务安装、独立原件备份与显式还原 |
| `src/modules/forzahorizon/effects.py` | Forza 专用 L2/R2 效果、优先级和跨帧状态 |
| `src/modules/forzahorizon/lighting.py` | 转速灯带、红线闪烁和挡位 Player LEDs 的传输无关状态 |
| `src/modules/haptics/` | 传输无关的握把混音与 PCM renderer，以及 USB audio、Bluetooth HD haptics 和 compatible fallback 路由 |
| `src/modules/dualsense/` | DualSense 枚举、有效输入驱动的不可变状态、电量、USB/BT handover、HID 报告、Bluetooth `0x36` 封包、自适应扳机和重连 |
| `src/modules/xinput/` | DualSense 输入映射、ViGEm Xbox 360 target、卡键保护、driver 探测和平台生命周期 |
| `src/modules/update/`、`packaging/windows/update_helper.py` | Windows 独立 EXE 的 Release 查询、SHA-256 校验、事务 journal、健康确认、恢复、Helper 调度和 R6 legacy bootstrap |
| `src/modules/dpi.py`、`packaging/windows/fhds.manifest` | Windows Per-Monitor v2 启动声明、运行时 bootstrap 和实际 DPI 状态查询 |
| `src/modules/dsx/` | DSX UDP 适配，只负责自适应扳机 |
| `src/modules/config/` | 默认设置、偏好文件、Profile 和路径规则 |
| `src/modules/gui/`、`src/modules/tui/` | 两套交互界面，设置能力应保持一致；FH6 文件工具分别从 `fh6_utilities_tab.py` 进入 |
| `tests/` | 行为、HID 字节、触觉、配置、发布和文档契约测试 |
| `packaging/`、`.github/workflows/release.yml` | ZUV、Windows EXE、Linux ELF、SHA-256 sidecar 和 Release 构建 |

## 常用命令

以下命令均从仓库根目录执行，除非命令中显式进入 `src`。

安装或同步开发环境：

```powershell
cd src
uv sync
```

从源码启动默认 GUI：

```powershell
cd src
uv run main.py
```

其他启动方式：

```powershell
cd src
uv run main.py --tui
uv run main.py --headless --debug
```

完整测试：

```powershell
uv run --project src --frozen pytest -q
```

基础检查：

```powershell
git diff --check
uv run --project src --frozen ruff check src tests packaging .github
uv run --project src --frozen pyrefly check src
uv run --project src --frozen python -m compileall -q src/main.py src/modules src/lang tests packaging/windows/update_helper.py packaging/windows/shortcut_links.py packaging/windows/write_sha256.py packaging/windows/dpi_runtime_hook.py
uv lock --check --project src
git status --short --branch
git diff --name-only
```

构建本地 ZUV：

```powershell
packaging\zuv\build_zuv.bat
```

构建带本仓库更新源的 ZUV，需要在 `cmd.exe` 中执行：

```bat
set UPDATE_REPO=piereacy/FH-DualSense-Enhanced
packaging\zuv\build_zuv.bat
```

构建 Windows EXE（生成 `FH-DualSense-Enhanced-R<n>.exe`、配套 `.sha256` 和更新 Helper）：

```powershell
packaging\windows\build_exe.bat
```

构建 Linux ELF：

```bash
bash packaging/linux/build_elf.sh
```

## 修改原则和限制

- 所有可调参数集中在 `src/modules/config/settings.py`。新增系统级设置时同步更新 `preferences.GLOBAL_FIELDS`；车辆手感参数默认应保持 Profile 级。
- GUI 和 TUI 的同类设置必须同时更新，并补齐 `src/lang/` 中所有非英语语言目录的翻译或明确回退行为。扳机与握把页面的字段归属只在 `src/modules/feedback_schema.py` 声明；GUI/TUI 只能渲染该共享 schema，不得各自复制一份分组表。扳机开关、常用调节和实验参数只出现在 `Trigger feedback`，握把开关、常用调节和实验参数只出现在 `Grip haptics`。
- Enhanced R4 的涡轮增压阻力、G 力阻力、L2/R2 碰撞扳机冲击和 L2/R2 空闲路面纹理属于实验性扳机反馈。六个开关及全部参数必须只出现在 GUI/TUI 默认折叠的“实验性功能”中，并继续默认关闭；未经新的产品决定不得移回普通“驾驶反馈”页面。
- 调整 R2 扳机键基础油门阻力或 G 力阻力前，必须先做 Enhanced R3/Enhanced R4 受控 A/B：固定车辆、路段、Profile、连接方式和游戏设置，记录 Forza 游戏内振动、Steam Input、最终效果来源、trigger mode 与 force。当前代码审计表明基础油门 ramp 未被 G 力层替换；不得仅凭“开启 G 力后手感相似”就删除旧路径、默认开启实验功能或改写默认参数。
- 总览状态必须来自线程安全的运行时快照或现有不可变快照，并由 GUI 主线程定时渲染；不得从日志文本反推状态，也不得只在控件创建时写占位值。UDP 状态只统计 324 字节有效包，错误主文案保持简短，详细原因放在提示或日志。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piereacy/FH-DualSense-Enhanced](https://github.com/piereacy/FH-DualSense-Enhanced) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
