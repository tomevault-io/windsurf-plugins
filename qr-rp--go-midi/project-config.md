---
trigger: always_on
description: GO_Midi 是一个 Windows 平台的 MIDI 自动演奏工具，主要用于游戏场景下的自动演奏。它能够解析 MIDI 文件并将音符转换为键盘按键，发送到目标游戏窗口。
---

# GO_Midi 项目上下文

## 项目概述

GO_Midi 是一个 Windows 平台的 MIDI 自动演奏工具，主要用于游戏场景下的自动演奏。它能够解析 MIDI 文件并将音符转换为键盘按键，发送到目标游戏窗口。

### 核心功能
- **MIDI 解析**：完整解析 MIDI 文件的所有音轨和通道，支持 Program Change 事件
- **智能移调**：自动计算最佳移调，让音符落在可用键位范围内
- **多通道支持**：8 个独立通道，每个通道可单独配置目标窗口、音轨和移调
- **键盘模拟**：向指定窗口发送按键事件
- **窗口恢复**：定时扫描窗口列表，自动恢复游戏重启后的窗口选择
- **播放列表**：多播放列表支持，拖拽排序，多种播放模式
- **AB 点循环**：右键设置循环区间，支持拖动调整
- **定时播放**：NTP 时间同步，精确定时开始播放
- **网络延迟补偿**：定时播放时提前触发，抵消网络延迟

### 技术栈
- **语言**：C++17
- **GUI 框架**：wxWidgets 3.3.1
- **构建系统**：CMake 3.10+
- **目标平台**：Windows 10/11 (64-bit)

---

## 项目结构

```
GO_Midi/
├── CMakeLists.txt          # CMake 构建配置
├── build_mingw.bat         # MinGW 构建脚本
├── src/
│   ├── App.cpp/h           # wxWidgets 应用入口
│   ├── resource.rc         # Windows 资源文件
│   ├── *.manifest          # Windows 清单文件
│   ├── assets/icon.ico     # 应用图标
│   ├── core/               # 核心引擎
│   │   ├── KeyboardSimulator  # 键盘模拟（Win32 API）
│   │   └── PlaybackEngine     # 播放引擎（线程、事件处理）
│   ├── midi/               # MIDI 解析
│   │   └── MidiParser         # MIDI 文件解析器
│   ├── ui/                 # 用户界面
│   │   ├── MainFrame          # 主窗口框架
│   │   ├── PlaybackState      # 播放状态机
│   │   ├── UIHelpers          # UI 辅助函数、UIConstants 常量
│   │   └── Widgets            # 自定义控件（ModernSlider, ScrollingText）
│   └── util/               # 工具类
│       ├── KeyManager         # 键位映射管理
│       ├── Logger             # 日志系统（线程安全）
│       ├── NtpClient          # NTP 时间同步客户端
│       └── PlaylistManager    # 多播放列表管理器
├── keymaps/                # 键位映射配置文件
│   └── 燕云十六声默认键位.txt
└── .github/workflows/      # CI/CD
    ├── dev.yml             # 开发分支构建
    └── release.yml         # Release 构建
```

---

## 构建指南

### 前置要求

1. **wxWidgets 3.3.1 源码**
   - 下载地址：https://github.com/wxWidgets/wxWidgets/releases/download/v3.3.1/wxWidgets-3.3.1.tar.bz2
   - 解压到项目根目录，确保路径为 `GO_Midi/wxWidgets-3.3.1/`

2. **编译器**
   - MinGW-w64（推荐）或 Visual Studio 2022

3. **CMake 3.10+**

### MinGW 构建（推荐）

```batch
# 运行构建脚本
build_mingw.bat

# 输出：build_mingw/GO_MIDI!.exe
```

### Visual Studio 构建

```batch
mkdir build && cd build
cmake .. -G "Visual Studio 17 2022" -A x64
cmake --build . --config Release

# 输出：build/Release/GO_MIDI!.exe
```

### CMake 优化配置

项目已针对体积优化进行配置：

- **MSVC**：`/O1 /GL /LTCG`（体积优先）
- **MinGW**：`-Os -flto -ffunction-sections -fdata-sections` + 链接时垃圾回收
- **静态链接**：禁用依赖库，生成独立可执行文件
- **std::filesystem**：MinGW 自动链接 `stdc++fs`（GCC 8 及以下需要）

---

## 核心模块说明

### PlaybackEngine（播放引擎）

位置：`src/core/PlaybackEngine.h/cpp`

- **线程模型**：独立播放线程 + 原子变量同步
- **事件处理**：预处理所有音符事件，按时间排序
- **防卡键**：`ActiveKeySet` 追踪活跃按键
- **智能移调**：基于音高直方图自动计算最佳移调
- **配置热更新**：通过版本号检测配置变化，实时重建事件列表

```cpp
// 关键接口
void load_midi(const Midi::MidiFile& midi_file);
void play() / pause() / stop() / seek(double time_s);
void set_channel_transpose(int channel, int semitones);
void set_pitch_range(int min_pitch, int max_pitch);
void notify_keymap_changed();  // 通知键位映射已更新
```

### MidiParser（MIDI 解析器）

位置：`src/midi/MidiParser.h/cpp`

- **支持格式**：MIDI Format 0/1
- **事件解析**：Note On/Off、Tempo、Time Signature、Program Change
- **时间转换**：Tick → 秒（支持变速）
- **音高统计**：每轨道音高直方图，全局直方图（排除打击乐 Channel 10 和低音乐器如 Bass）
- **中文路径**：使用 `std::filesystem::path` 支持 Unicode 路径

```cpp
// 核心数据结构
struct RawNote {
    float start_s;      // 开始时间（秒）
    int pitch;          // MIDI 音高 (0-127)
    float duration;     // 持续时间
    int track_index;    // 轨道索引
    int channel;        // MIDI 通道 (0-15)
    int velocity;       // 力度 (0-127)
    int program;        // 乐器编号 (0-127)
};
```

### KeyboardSimulator（键盘模拟）

位置：`src/core/KeyboardSimulator.h/cpp`

- **实现方式**：Win32 `SendInput` API
- **窗口选择**：枚举所有可操作窗口
- **按键发送**：支持普通键和修饰键（Shift/Ctrl）
- **扫描码缓存**：静态缓存减少系统调用

```cpp
// 窗口信息结构
struct WindowInfo {
    HWND hwnd;
    std::string title;
    std::string process_name;
    unsigned long pid;
};
```

### KeyManager（键位管理）

位置：`src/util/KeyManager.h/cpp`

- **映射格式**：`音符: 按键`（支持音名或 MIDI 编号）
- **音名支持**：升号（C#, F#）和降号（Db, Eb）均支持
- **性能优化**：O(1) 查找缓存数组
- **配置持久化**：加载/保存键位配置文件
- **中文路径**：使用 `std::filesystem::path` 支持 Unicode 路径

```cpp
// 键位映射结构
struct KeyMapping {
    int vk_code;     // Windows 虚拟键码
    int modifier;    // 修饰键：0=无, 1=Shift, 2=Ctrl
};
```

### UIHelpers（UI 辅助模块）

位置：`src/ui/UIHelpers.h/cpp`

- **UIConstants 命名空间**：集中管理 UI 字符串常量，避免重复定义
- **UIHelpers 类**：UI 更新辅助函数
- **ChannelUIUpdater 类**：批量更新通道控件

```cpp
// 字符串常量定义
namespace UIConstants {
    const wxString DEFAULT_WINDOW = "未选择";
    const wxString DEFAULT_TRACK = "全部音轨";
    const wxString DEFAULT_KEYMAP = "默认键位";
    const wxString MODE_SINGLE = "单曲播放";
    const wxString MODE_SINGLE_LOOP = "单曲循环";
    const wxString MODE_LIST = "列表播放";
    const wxString MODE_LIST_LOOP = "列表循环";
    const wxString MODE_RANDOM = "随机播放";
}
```

### 窗口恢复机制

位置：`src/ui/MainFrame.cpp` - `TryRecoverWindows()`

- **触发时机**：每 5 秒定时扫描（仅在非播放状态）
- **恢复策略**：按窗口标题精确匹配
- **工作流程**：
  1. 刷新窗口列表 `UpdateWindowList()`
  2. 遍历所有通道，检查是否为"未选择"状态
  3. 读取配置中保存的 `WindowTitle`
  4. 调用 `FindWindowByTitle()` 精确匹配标题
  5. 匹配成功则恢复窗口选择

```cpp
// 核心辅助函数
int FindWindowByTitle(const wxString& title);  // 按标题查找窗口索引
```

### PlaybackStateMachine（播放状态机）

位置：`src/ui/PlaybackState.h/cpp`

- **状态定义**：Idle, Playing, Paused, Stopped, Scheduled, Error
- **状态转换**：通过回调通知 UI 更新
- **字符串缓存**：减少 UI 更新时的内存分配

---

## 开发规范

### 代码风格


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qr-rp/GO_Midi](https://github.com/qr-rp/GO_Midi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
