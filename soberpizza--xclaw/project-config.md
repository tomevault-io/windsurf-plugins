---
trigger: always_on
description: 纯视觉、纯键鼠的 Windows CUDA 桌面代理框架。模拟真人使用电脑的完整认知回路：
---

# X-Claw

纯视觉、纯键鼠的 Windows CUDA 桌面代理框架。模拟真人使用电脑的完整认知回路：

**截屏（眼睛）→ YOLO + RapidOCR + Florence-2 感知（视觉皮层）→ 结构化 JSON（语言区）→ OS 原生键鼠操作（手）**

感知层将屏幕像素转化为带编号的元素列表（纯文本 JSON），外部 Agent 的 LLM 仅消费该文本做决策，不接触任何图像数据。

仅支持 Windows (CUDA NVIDIA GPU)。

## 核心约束（不可违背）

X-Claw 是一个数字人的感官和手脚。以下约束定义了项目的身份边界，任何违反均视为架构缺陷：

1. **唯一信息源是屏幕像素。** 不读取 DOM、不调用无障碍树（Accessibility API）、不使用浏览器 DevTools / CDP、不做进程间内存读取。一切状态信息必须从截屏中重建。

2. **唯一输出通道是 OS 原生键鼠事件。** Windows 通过 ctypes SendInput。不注入 JavaScript、不调用应用程序 API。复制粘贴（Ctrl+C/V）属于真人正常操作，Agent 可以使用；但禁止将剪贴板作为绕过键鼠的隐蔽数据传输通道。

3. **感知层输出纯文本，LLM 不接触图像。** 感知管线的最终产物是结构化 JSON（元素 ID、bbox、类型、文本内容），外部 Agent 的 LLM 仅基于该文本做推理和决策。不向 LLM 传递截图、不依赖多模态能力、不引入"让 LLM 看一眼图片辅助判断"的回退路径。视觉理解的全部责任由感知层承担。

4. **服务单个 Agent。** 架构为单 Agent 独占设计，不考虑多 Agent 并行、分布式调度、远程 RPC。

5. **感知层与执行层通过协议解耦，但不引入中间层语义推理。** 感知层输出带编号标注的元素列表，所有高层语义理解（意图识别、任务规划）由外部 LLM 在纯文本域完成。禁止在管线中插入"UI 语义分类器"、"控件类型推断器"、"意图预测器"等中间智能层。

6. **反检测是一等需求。** HumanizeStrategy 不是可选的锦上添花，而是与感知、执行同等重要的核心能力。任何新增的操作路径都必须经过人性化策略层，不允许存在绕过 HumanizeStrategy 的快捷通道。

## 开发环境

- Python 3.12（通过 `.python-version` 锁定）
- 包管理：[uv](https://docs.astral.sh/uv/)
- 安装依赖：`uv sync`（PyTorch 从 `download.pytorch.org/whl/cu121` 安装）
- 运行命令：`uv run xclaw <command>`

## 全局安装

在项目目录下执行：
```bash
uv tool install --editable . --python 3.12
```
安装后可在任意路径使用 `xclaw` 命令。卸载：`uv tool uninstall xclaw`。

## 关键路径

```
xclaw/
├── __init__.py            # 包入口
├── config.py              # 全局配置（路径、感知参数、人性化参数）
├── cli/                   # Click CLI 子包
│   ├── __init__.py        # cli group 入口 + 命令注册
│   ├── core.py            # setup() + output() + IS_DEBUG
│   ├── _silence.py        # 第三方库静默逻辑
│   └── commands/
│       ├── __init__.py
│       ├── look.py        # look 命令（直接调用 PerceptionEngine）
│       └── action.py      # click/type/press/scroll/wait 命令
├── platform/
│   ├── __init__.py        # 导出 PLATFORM / PERCEPTION_CONFIG 单例
│   ├── detect.py          # 平台检测（内存、GPU — Windows only）
│   └── gpu.py             # 感知引擎硬件配置（CUDA + TRT / CPU）
├── core/
│   ├── __init__.py
│   ├── screen.py          # 截屏（mss）
│   ├── cache.py           # LRU 感知缓存
│   ├── pipeline.py        # 视觉管线：感知 + 坐标排序
│   └── perception/
│   │   ├── __init__.py
│   │   ├── backend.py     # PerceptionBackend 协议（抽象接口）
│   │   ├── engine.py      # PerceptionEngine 单例：委托 backend 编排
│   │   ├── pipeline_backend.py # 默认后端：YOLO + OCR + Florence-2
│   │   ├── omniparser.py  # OmniDetector（YOLO 双后端：ONNX / ultralytics）
│   │   ├── icon_classifier.py # Florence-2 图标描述（OmniParser V2 fine-tuned）
│   │   ├── ocr.py         # RapidOCR (PPOCRv5 mobile ONNX, CUDA)
│   │   ├── merger.py      # IoU 去重 + YOLO/OCR 空间融合
│   │   └── types.py       # RawElement / TextBox 数据类型
├── action/
│   ├── __init__.py        # ActionBackend 单例（get_backend/set_backend）+ 向后兼容函数
│   ├── backend.py         # ActionBackend 协议（抽象接口）
│   ├── native_backend.py  # NativeActionBackend：委托 win32 模块 + HumanizeStrategy
│   ├── dry_run_backend.py # DryRunBackend：记录动作不触发 OS 事件（测试用）
│   ├── humanize_strategy.py # HumanizeStrategy 协议 + NoopStrategy / BezierStrategy
│   ├── humanize.py        # bezier_point() 纯数学工具函数
│   ├── mouse.py           # 高层点击/滚动/拖拽接口（委托 ActionBackend）
│   ├── keyboard.py        # 高层打字/按键/组合键接口（委托 ActionBackend）
│   ├── mouse_win32.py     # Windows: ctypes SendInput 鼠标（raw 操作 + 机械延迟）
│   └── keyboard_win32.py  # Windows: ctypes SendInput 键盘（raw 操作 + 机械延迟）
├── installer/
│   ├── __init__.py        # 安装工具包
│   ├── postinstall.py     # 安装后初始化（目录结构 + 模型下载 + init）
│   └── download_gui.py    # Tkinter 模型下载 GUI
├── debug/
│   ├── __init__.py
│   └── pipeline.py        # 调试用管线可视化
└── skills/
    └── SKILL.md           # Claude Code 技能入口
scripts/
├── download_models.py     # 模型下载（OmniParser V2）
├── export_yolo_onnx.py    # YOLO .pt → .onnx 导出
├── build_installer.py     # 安装包构建
└── installer.iss          # Windows Inno Setup 脚本
```

## 硬件配置

| 组件 | CUDA 模式 | CPU 回退 |
|------|-----------|----------|
| 键鼠控制 | ctypes `SendInput` | ctypes `SendInput` |
| YOLO 检测 | TensorRT EP / CUDA EP / ultralytics | CPU EP / ultralytics |
| OCR | RapidOCR PPOCRv5 mobile CUDA | RapidOCR PPOCRv5 mobile CPU |
| 图标描述 | Florence-2 FP16 CUDA | Florence-2 FP32 CPU |
| torch 索引 | `pytorch-cu121` | `pytorch-cu121` |

- `xclaw/platform/detect.py` 检测内存和 GPU
- `xclaw/platform/gpu.py` 根据 GPU 生成 `PerceptionConfig`（含 TRT 开关）
- `xclaw/action/__init__.py` 通过 `ActionBackend` 单例路由，`set_backend()` 可注入自定义后端

### PyTorch 包索引

项目通过 `[tool.uv.index]` + `[tool.uv.sources]` 配置了自定义索引：

- **pytorch-cu121** — PyTorch CUDA 12.1 wheels

OCR 使用 RapidOCR（PaddleOCR ONNX 封装），通过已有的 `onnxruntime-gpu` 走 `CUDAExecutionProvider`，无需 `paddlepaddle-gpu`。

## 环境变量

| 变量 | 说明 | 默认 |
|------|------|------|
| `XCLAW_HUMANIZE` | 设为 `1` 启用人性化鼠标移动和打字延迟 | `0` |
| `XCLAW_HOME` | 项目根目录路径（仅非 editable 安装时需要） | 自动推算 |
| `XCLAW_DATA` | 用户可写数据目录（截图、日志、模型） | 开发模式=PROJECT_ROOT，安装模式=`%LOCALAPPDATA%\X-Claw` |
| `XCLAW_TRT` | 设为 `0` 禁用 TensorRT EP（回退 CUDA EP） | `1` |
| `DEBUG` | 设为 `1` 启用 CLI 调试日志输出到 stderr | `0` |

## 感知引擎

- `PerceptionEngine`（`perception/engine.py`）是感知层单例，委托 `PerceptionBackend` 协议：
  - 默认后端 `PipelineBackend`（`pipeline_backend.py`）组合三个子模块：
    - `OmniDetector`：YOLO icon_detect，优先 TensorRT EP，回退 CUDA EP，再回退 ultralytics
    - `OCREngine`：RapidOCR PPOCRv5 mobile 中英双语 (CUDA)
    - `IconClassifier`：Florence-2 图标描述（OmniParser V2 fine-tuned Florence-2-base）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SoberPizza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
