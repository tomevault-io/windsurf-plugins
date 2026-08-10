---
trigger: always_on
description: Windows / Linux 桌面应用 + Android 客户端：实时 AI 音频降噪 / 目标说话人提取 / 回声消除，支持本地麦克风和远程网络推流。
---

# PureVox — AI 麦克风降噪

Windows / Linux 桌面应用 + Android 客户端：实时 AI 音频降噪 / 目标说话人提取 / 回声消除，支持本地麦克风和远程网络推流。

**栈**: Python 3.8+（最低；Win7 需 3.8）+ PySide6 + 纯 C 共享库（gcc/mingw 编译，ctypes 绑定）+ ONNX Runtime（==1.11.1，模型 opset 13/14/15，均 ≤16）
**桌面入口**: `python run_pyside6.py`
**Android 入口**: `android/` — Kotlin + OkHttp + Opus JNI

---

## 维护准则（发版后管理，所有贡献者必读）

**功能最小化模型** —— 本项目的首要约束：

1. **一个功能只有一条规范实现路径**。禁止"功能 ABC 三种都能用"的设计——多套平行实现等于高维护成本。新增功能有多个可行做法时，只保留一种并写进文档，其余不进入代码。
2. **先扩展，再新建**。开新方法 / 新类 / 新文件之前，先搞清楚已有方法能否扩展：优先 改已有函数/类 → 加参数/加配置 → 复用既有抽象；确认确实无法扩展才允许新建，并在 CHANGELOG 说明为何不能扩展。
3. **被替代的实现不保留平行代码**。如 Linux 的 PortAudio/GStreamer/JACK、旧虚拟麦克风架构等已弃用方案，直接删除，不留"备选"。
4. **改动前先读对应模块，尊重既有设计意图**；删除功能需在 CHANGELOG 记录。

**本项目的单一实现路径（强制执行）**：

- Linux 音频采集/输出**只用原生 PipeWire**（`pvpipe`）
- 虚拟麦克风（Linux）= 单一生产者 + 双出口，全部健康，详见下方「Linux 音频架构」：
  ① 单声道 null-sink `purevox_out`（唯一写入口）；② 内置 monitor
  `purevox_out.monitor`（宽口径源）+ 非 monitor 真源 `purevox_mic`
  （`module-remap-source` 把 monitor 重映射而来，供 OBS 等"只列真源"软件）。
  不用 pw-loopback。**禁建第二路源用 `module-null-sink media.class=Audio/Source/Virtual`
  ——实测会把 pipewire-pulse 协议搞坏（pactl 报协议错误、plasma-pa context kaput、
  系统托盘清空，仅重启 pipewire-pulse 恢复）**，健康方案是 module-remap-source
- 音频格式一律 **F32 单声道 48kHz**（PipeWire 负责重采样与声道转换，模型永远拿 48k 单声道）
- 设备枚举只用 `pw-dump` 标准 introspection（`pvplatform.audio.pwpipe_client`）

---

## 运行 / 构建

**内嵌 Python 3.8（推荐，独立于系统环境）**：本项目可自带独立 Python 3.8，
与系统 Python（如 3.14）完全隔离。Windows 走 NuGet 下载预编译包；Linux 无预编译
3.8 可下，源码以 **git 子模块** `packages/cpython`（CPython@v3.8.20）锁定，
由引导脚本 out-of-tree 一次性编译。产物统一放 `packages/`。

- 克隆后先 `git submodule update --init --depth 1 packages/cpython` 拉子模块
- `./bootstrap_python38.sh`（Linux，幂等）→ 生成自包含 `packages/python38/` + 装依赖
- `./bootstrap_python38.ps1`（Windows）→ 生成 `packages\python38w\`（NuGet 完整版，含头文件/链接库）
- 内嵌解释器与系统 Python 互相独立；`packages/python38*`、`.py38-src/` 不进版本库（gitignore）

### Windows (PowerShell)

```powershell
chcp 65001
# 方式一（内嵌 3.8，推荐）：
powershell -ExecutionPolicy Bypass -File bootstrap_python38.ps1
# 方式二（系统 Python）：pip install -r requirements.txt -r requirements-win.txt
python run_pyside6.py
powershell -ExecutionPolicy Bypass -File build_win.ps1   # 打包产物目录 dist/PureVox/（自动用 packages\python38w\python.exe）
# 注：Windows 侧 aimic.dll 用 mingw gcc 编译（setup.py 走 CC 或 PATH 上的 gcc，
# 链接捆绑的 onnxruntime-win-x64-1.11.1）
```

### Windows 7 兼容性（实测结论，勿回退）

纯 PySide6 6.1.3 包无法直接跑 Win7——Qt 6.2+ 官方仅 Win10+，6.6.x import 即报
`DLL load failed ... 找不到指定的程序`；**PySide6==6.1.3 是最后一个支持 Win7 的版本**
（requirements.txt 已锁死并注释原因）。另外两个 Win7 缺失项必须在打包时补：

- **API-Set 转发 DLL**：捆绑的 onnxruntime.dll 还导入 `api-ms-win-core-libraryloader-l1-2-0.dll`
  和 `api-ms-win-core-processtopology-obsolete-l1-1-0.dll`（Win8+ 的 API-Set 由内核虚拟解析，
  Win7 与其构建机 System32 均无物理文件）。仓库在
  `packages/onnxruntime-win-x64-1.11.1/lib/` 固化两个 **x64 转发 stub**（导出符号转发到
  KERNEL32；生成材料见其下 `apiset/*.def`，用 mingw `x86_64-w64-mingw32-gcc -shared`
  复现，例如 `x86_64-w64-mingw32-gcc -shared stub.c apiset/libloader.def -o
  api-ms-win-core-libraryloader-l1-2-0.dll`）。`build_win.ps1` 打包时从仓库拷这两个
  stub 进 `_internal`，勿改回"从构建机 System32 拷"。
- **MSVC 运行库**：onnxruntime 依赖 MSVCP140/VCRUNTIME140 等，`build_win.ps1` 会把构建机
  System32 的 VC runtime 拷进包，避免 Win7 需单独装 VC++ redist。
- **打包瘦身勿删 `Qt6Qml.dll`/`Qt6Quick.dll`**：PySide6 核心库 `pyside6.abi3.dll`（所有
  Qt*.pyd 都链接它）**硬依赖 `Qt6Qml.dll`**，`build_win.ps1` 第 4 步若删除它，EXE 在 Win7
  启动即报 `DLL load failed while importing QtWidgets: 找不到指定的模块`（2026-08-09 实机
  pefile 分析确认；与 pyinstaller/hooks-contrib 版本无关）。瘦身只允许删 import 闭包外的
  `Qt6Pdf.dll`/`Qt6DataVisualization.dll`。

注意：四件套 wheel 名含 `abi3`；在线安装 PySide6==6.1.3 时会自动带对版本。

**Qt 6.5+ API 禁忌（Win7 实测 2026-08-09）**：`QStyleHints::colorScheme()` 与
`Qt.ColorScheme` 是 Qt 6.5+ 才有，Win7 锁定的 PySide6 6.1.3 上没有——启动即抛
`AttributeError: 'QStyleHints' object has no attribute 'colorScheme'`。深色判定一律用
**调色板亮度**（`app.palette().window().color().lightness() < 128`，Qt 6.1 即可用），
已固化在 `theme_colors.is_dark_current()`（2014 版），UI 主题同步 `_sync_theme_ui`
也走它。新增代码禁写 `styleHints().colorScheme()`。

**`.ps1` 脚本必须纯 ASCII（英文）**：`build_win.ps1` / `bootstrap_python38.ps1`
不含中文/非 ASCII/BOM。Windows PowerShell 5.1 对无 BOM 的 UTF-8 脚本按 ANSI
(cp1252/GBK) 误读导致语法错误（`chcp 65001` 只在本机掩盖）；中文文件名
（如 `用户手册.html`）在脚本里用通配符（`*.html`）引用，不写字面量。

### Linux

依赖因发行版而异（Ubuntu / Fedora / AOSC 包名不同，参考 `.github/workflows/ci.yml` 与 README）。AOSC 示例：

```bash
sudo oma install -y gcc pkgconf pipewire libpipewire-0.3-devel
# 内嵌 3.8（推荐）：
./bootstrap_python38.sh
./py38 setup.py build_ext --inplace --force   # 产出 libaimic.so + libpvpipe.so
./py38 run_pyside6.py
bash pack_deb.sh                              # deb → dist/PureVox-Linux-x64-<date>-release.deb
bash pack_rpm.sh                              # rpm → dist/PureVox-Linux-x64-<date>-release.rpm
bash pack_appimage.sh                         # AppImage → dist/PureVox-Linux-x64-<date>-release.AppImage
```

deb 布局：`/opt/purevox/` 放全部源码+libaimic.so/libpvpipe.so+模型+html+捆绑的 `libonnxruntime.so*`（1.11.1）；
`/usr/bin/purevox` 启动脚本（先导出 `LD_LIBRARY_PATH=/opt/purevox` 再 exec）。
`/usr/share/applications/purevox.desktop` + hicolor 图标。Depends 按 AOSC 包名（无 onnxruntime，
PT 已捆绑）。`server/opus.dll`
是 Windows 的，不入 deb。`.so` 为固定名 `libaimic.so`/`libpvpipe.so`，不用 `sysconfig.EXT_SUFFIX` 定位。
Linux 输入/输出/设备枚举/AEC 全原生 PipeWire；opuslib 缺失时 `pip install --user`（写进 Recommends）。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a2heng/PureVox](https://github.com/a2heng/PureVox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
