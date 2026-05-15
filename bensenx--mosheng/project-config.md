---
trigger: always_on
description: 声音，化为笔墨。Windows 本地智能语音输入工具，基于 Qwen3-ASR-1.7B。支持**按住录音**和**按键切换**两种模式，松开/再按后自动识别并粘贴文本。支持**渐进式输入**（停顿时自动输入已识别文本）。暗色毛玻璃 Glassmorphism UI 风格。
---

# MoSheng (墨声)

声音，化为笔墨。Windows 本地智能语音输入工具，基于 Qwen3-ASR-1.7B。支持**按住录音**和**按键切换**两种模式，松开/再按后自动识别并粘贴文本。支持**渐进式输入**（停顿时自动输入已识别文本）。暗色毛玻璃 Glassmorphism UI 风格。

## 运行

```bash
uv run --project E:\VoiceInput python E:\VoiceInput\main.py
```

## 环境

- **包管理**: UV (`pyproject.toml` + `uv.lock`)，不使用 requirements.txt
- **Python**: 3.12-3.13（3.14 无 PyTorch CUDA wheels）
- **UI 框架**: PySide6 (Qt)，暗色毛玻璃 Glassmorphism 主题 (DWM Acrylic backdrop)
- **PyTorch CUDA**: `torch` 必须作为直接依赖才能让 `[tool.uv.sources]` 指向 cu128 索引
- **GPU**: RTX 5090 (Blackwell/sm_120) 需要 cu128 索引；cu124 仅支持到 sm_90
- **依赖覆盖**: `override-dependencies` 强制 `numba>=0.60`, `llvmlite>=0.43`, `librosa>=0.10`（qwen-asr 传递依赖版本过低）

## 项目结构

```
main.py                 入口：单实例互斥锁 → QApplication → 加载界面 → 环境检查 → 模型加载 → 事件循环
config.py               默认配置常量（含 input_device）
i18n.py                 国际化：73 个翻译键（zh/en），tr() 查找 + 系统语言自动检测
settings_manager.py     用户设置持久化 (~/.mosheng/settings.json)
pyproject.toml          UV 项目配置（依赖、CUDA 索引、构建）
launcher.py             分发包入口（仅 stdlib，PyInstaller 编译为 MoSheng.exe）
_setup.cmd              首次运行安装脚本（GPU 检测、镜像选择、uv sync）
assets/
  shaders/smoke.frag    GLSL 片段着色器源码（五色冷墨韵波形，中心对称 cos 波）
  shaders/smoke.frag.qsb  pyside6-qsb 编译产物
core/
  asr_base.py           ASR 抽象基类 (ABC)，可替换模型
  asr_qwen.py           Qwen3-ASR-1.7B 实现（含音频诊断日志）
  audio_recorder.py     sounddevice 录音，16kHz 单声道 float32，支持指定设备，drain_buffer + EMA RMS + VAD feed
  vad.py                Silero VAD V5 封装：torch.hub 加载，CPU 推理，512 样本/chunk，语音活动概率检测
  speaker_verifier.py   声纹识别：SpeechBrain ECAPA-TDNN 两级验证（快速路径/慢速分段）
  text_injector.py      剪贴板写入 + ctypes SendInput 模拟 Ctrl+V，hotkey-aware 修饰键释放
  hotkey_manager.py     WH_KEYBOARD_LL 钩子级按键抑制 + VK 码分组匹配，支持 push_to_talk / toggle
  key_suppression_hook.py  ctypes 低级键盘钩子，选择性抑制物理按键，放行 SendInput 注入事件
ui/
  app.py                QSystemTrayIcon + WorkerThread 组件协调器（核心调度）
  splash_screen.py      启动加载界面（glassmorphism，模型加载期间显示）
  overlay_window.py     QQuickView + GLSL shader 悬浮窗（五色冷墨韵波形），FFT 5 频段驱动，click-through
  overlay.qml           ShaderEffect 承载，暴露 amplitude/stateBrightness/stateHue/b0-b4 属性
  shaders/smoke.frag.qsb  编译后的 GLSL 片段着色器（pyside6-qsb 生成）
  enrollment_dialog.py  声纹注册引导对话框（3 段录音 + 背景线程处理）
  settings_window.py    QDialog 设置界面（Glassmorphism + DWM Acrylic backdrop）
  styles.py             Glassmorphism QSS + 颜色常量 + ToggleSwitch + load_icon_pixmap + draw_section_icon
utils/
  logger.py             日志配置
configs/
  pyproject-cuda.toml   CUDA 版 pyproject（cu128 索引）
  pyproject-cpu.toml    CPU 版 pyproject（PyPI torch）
  uv-cuda.lock          CUDA 版锁文件
  uv-cpu.lock           CPU 版锁文件
scripts/
  build_dist.py         构建分发包（PyInstaller + 源码复制 + uv.exe）
```

## 设置项

| 分类 | 项目 | 运行时热更新 |
|------|------|:---:|
| 快捷键 | 按键组合、录音模式（按住/切换）、渐进式输入、静音时长 | ✓ |
| 语言 | 界面语言（中文/English） | ✗ 需重启 |
| 语音识别 | ASR 模型、GPU 设备 | ✗ 需重启 |
| 音频输入 | 麦克风设备选择 | ✓ |
| 输出 | 提示音、悬浮窗、剪贴板恢复 | ✓ |
| 词汇 | 自定义词汇表、CSV/TXT 导入 | ✓ |
| 声纹识别 | 启用/禁用、注册声纹、阈值配置 | ✓ 懒加载/卸载 |

## 分发包

### 构建

```bash
uv run python scripts/build_dist.py
```

产出 `dist/MoSheng/`（~70MB），包含：
- `MoSheng.exe` — PyInstaller 编译的 launcher（仅 stdlib，~7MB）
- `uv.exe` — 包管理器（~57MB）
- 源码（main.py, core/, ui/, utils/, assets/）
- `_setup.cmd` + `configs/` — 首次运行安装

### 用户运行流程

1. 双击 `MoSheng.exe`
2. Launcher 检查 `.venv/.mosheng_version`，不匹配则运行 `_setup.cmd`
3. `_setup.cmd`：GPU 检测 → 选择 CUDA/CPU pyproject.toml → `uv sync`
4. Launcher 启动 `pythonw.exe main.py`（无控制台）

### 分发包注意事项

- `_setup.cmd` 必须纯 ASCII（cmd.exe 用系统代码页 GBK 解析，UTF-8 中文会错位）
- `if errorlevel 1 goto :label` 比 `if !ERRORLEVEL! NEQ 0` 更可靠
- 版本更新时同步 `launcher.py:CURRENT_VERSION` 和 `_setup.cmd` 写入的版本号

## 线程模型

- **主线程**: `QApplication.exec()` 事件循环，拥有所有 QWidget 和 QSystemTrayIcon
- **KeySuppressionHook 线程**: WH_KEYBOARD_LL 钩子 + 消息泵，VK 码分组匹配触发 start/stop → 写入 worker cmd_queue
- **WorkerThread (QThread)**: 从 cmd_queue 读命令，驱动录音→声纹验证→识别→粘贴流程，通过 `state_changed` 信号更新 UI；渐进模式下 50ms 轮询 Silero VAD 检测停顿
- **PortAudio 回调线程**: 音频帧写入 buffer

## 编码注意事项

### ctypes SendInput
- INPUT 结构体 union 必须包含 MOUSEINPUT + KEYBDINPUT + HARDWAREINPUT 三个成员，否则 `sizeof(INPUT)` 为 24（应为 40），`SendInput` 会静默失败
- `keyboard.send()` 底层用已废弃的 `keybd_event`，在 Win11 记事本和 cmd 中不可靠；用 ctypes `SendInput` 代替
- `_send_ctrl_v(hotkey_vks)` 接收快捷键 VK 集合，跳过被钩子抑制的按键（OS 不认为它们被按下），仅释放非快捷键修饰键
- Cursor/VS Code 内嵌终端 (xterm.js) 对 SendInput Ctrl+V 不响应，属环境限制

### WH_KEYBOARD_LL 钩子 (key_suppression_hook.py)
- 钩子回调必须在有消息泵的线程上运行（`GetMessageW` 循环），否则收不到事件
- `SetWindowsHookExW` 的 argtypes/restype 必须显式声明为 `c_void_p`，默认 `c_int` 在 64 位系统上截断句柄
- 通过 `LLKHF_INJECTED` 标志区分物理按键和 SendInput 注入事件，注入事件始终放行
- 回调返回 1 抑制事件（不调用 `CallNextHookEx`），返回 `CallNextHookEx(...)` 放行
- 快捷键在钩子层面被吞掉 → OS 完全看不到 → 无 Win 键开始菜单、无截图、无剪贴板历史等副作用

### 快捷键 VK 码分组匹配
- 每个按键名映射为一组 VK 码：如 "ctrl" → `{0x11, 0xA2, 0xA3}`（通用 + 左 + 右）
- 匹配条件：每个组中至少有一个 VK 被按下（`_all_groups_pressed()`），而非要求所有 VK 同时按下
- VK 码表来自 `keyboard._winkeyboard.official_virtual_keys`，仅读取数据不使用其钩子

### PySide6 / Qt
- `QApplication.setQuitOnLastWindowClosed(False)` — 托盘应用必须设置，否则关闭设置窗口会退出程序
- 跨线程 UI 更新用 Qt 信号/槽（自动 `QueuedConnection`）或 `QMetaObject.invokeMethod`
- Overlay 已从 QWidget 迁移到 QQuickView + GLSL ShaderEffect（GPU 渲染）
- Overlay click-through 用 `winId()` 直接获取 HWND + Windows API `SetWindowLongW`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bensenx/MoSheng](https://github.com/bensenx/MoSheng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
