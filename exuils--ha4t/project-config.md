---
trigger: always_on
description: HA4T (Hybrid App For Testing Tool) — 跨平台 UI 自动化测试框架，支持 **Android**、**iOS**、**HarmonyOS NEXT** 和 **Web (CDP)**。基于 uiautomator2、facebook-wda、hmdriver2、PaddleOCR 和 OpenCV 构建。支持图像识别、OCR 文字识别、原生控件定位和 WebView 调试四种定位方式。
---

# Repository Guidelines

## Project Overview

HA4T (Hybrid App For Testing Tool) — 跨平台 UI 自动化测试框架，支持 **Android**、**iOS**、**HarmonyOS NEXT** 和 **Web (CDP)**。基于 uiautomator2、facebook-wda、hmdriver2、PaddleOCR 和 OpenCV 构建。支持图像识别、OCR 文字识别、原生控件定位和 WebView 调试四种定位方式。

发布在 PyPI 上，包名 `ha4t`，当前版本 0.1.6。参见 `pyproject.toml`。

## Architecture & Data Flow

### 分层架构

```
┌────────────────────────────────────────────┐
│  Public API   (ha4t/__init__.py)             │
│  connect() → Device  |  include()  | Selector│
└──────────────────┬─────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  Device  (ha4t/device/)                       │
│  InteractionMixin  QueryMixin  AppMixin      │
│  FileMixin  CaptureMixin  DeviceBase (state) │
│  _resolve_selector → to_android/to_ios/etc.  │
│  _find_pos_*  → matchers                     │
└──────┬────────────────────┬──────────────────┘
       │                    │
┌──────▼──────┐  ┌─────────▼──────────────────┐
│ Drivers     │  │ Matchers (driver-agnostic)  │
│ BaseDriver  │  │ find_pos_by_image           │
│ Android     │  │ find_pos_by_ocr             │
│ iOS         │  │ click_inside_template       │
│ Harmony     │  └─────────┬──────────────────┘
└─────────────┘       ┌────┴──────┐
                 ┌────▼───┐ ┌────▼────┐
                 │ aircv   │ │ OCR     │
                 │ cv.py   │ │PaddleOCR│
                 └─────────┘ └─────────┘
┌──────────────────────────────────────────────┐
│ CDP Subsystem  (ha4t/cdp/)                     │
│ CdpServer → Page → Element (WebView via WS)   │
└──────────────────────────────────────────────┘
```

### 数据流

1. **connect(platform, ...)** → 构建 `BaseDriver` 子类 → 包装为 `Device`（Mixin 多重继承）
2. **dev.click(Selector)** → `DeviceBase._resolve_selector()` → 按平台提取原生 kwargs → 按 args[0] 类型分派到 OCR/图像/原生驱动
3. **dev.click(text="登录")** → `_resolve_selector()` 调用 `to_native(kwargs, platform)` 做跨平台字段映射 → 驱动层 `find(**kwargs)`
4. **dev.wait(text, ...)** → 轮询 `_exists()` → OCR 或驱动查找 → 超时抛 `ElementWaitTimeoutError`
5. **POM 文件读写**：`pom/<page>.py` (Python 源码) → `_parse_pom_py()` (ast + tokenize) → `ElementShape` dict → `_render_pom_py()` → 确定性 Python 源码（git-diff 稳定）

### 重要设计模式

- **Mixin 组合 Device**：`Device` 无自身逻辑，纯粹由 5 个 Mixin + DeviceBase 通过 MRO 装配
- **Selector 不可变值对象**：`__slots__` + `__setattr__` 禁止修改，所有转换方法返回新实例；`repr()` eval-safe 支持 POM 文件往返
- **screenshot_fn 依赖注入**：Matchers 接收 `screenshot_fn: Callable`（driver.screenshot 的绑定方法），与驱动完全解耦
- **惰性 OCR 加载**：`matchers/ocr.py` `_ocr = None` + `_get_ocr()` 首次调用才导入 PaddleOCR（避免慢导入）
- **@cost_time 横切装饰器**：剥离 `_` 前缀元 kwargs、记录耗时、包装 allure.step、失败时截图
- **CDP 异步隔离**：`cdp/cdp.py` 在守护线程中运行 async WebSocket 循环，公共 `Page` API 全同步

## Key Directories

| 目录 | 用途 |
|---|---|
| `ha4t/` | 核心框架包 |
| `ha4t/device/` | Device 类（Mixin 装配）— interaction, queries, apps, files, capture |
| `ha4t/drivers/` | 平台驱动 — base.py (ABC), android.py, ios.py, harmony.py |
| `ha4t/matchers/` | 找点策略 — image.py (OpenCV), ocr.py (PaddleOCR) |
| `ha4t/selector.py` | Selector 类 + 跨平台字段映射 (to_android/to_ios/to_harmony) |
| `ha4t/cdp/` | WebView CDP 自动化 — cdp.py, server.py, by.py |
| `ha4t/config.py` | DeviceConfig + GlobalConfig 单例 |
| `ha4t/exceptions.py` | 异常层次 (HA4TError → 9 个子类) |
| `ha4t/orc.py` | PaddleOCR 封装 |
| `ha4t/aircv/` | 基于 OpenCV 的图像匹配（forked from airtest） |
| `ha4t/editor/` | FastAPI + Vue 3 UI 编辑器（本地开发服务器） |
| `ha4t/editor/skills/` | 工作区初始化模板（AGENTS.md / conftest.py / pyproject.toml / README.md） |
| `tests/` | 测试套件（unittest.TestCase，75 个测试） |
| `docs/` | Sphinx 文档 (sphinx_rtd_theme, zh_CN) |
| `.github/workflows/` | CI — publish (PyPI), docs-and-deploy, test (stale), welcome |

## Development Commands

```bash
# 安装依赖
uv sync

# 运行测试
uv run pytest tests/ -v

# 启动编辑器
uv run uvicorn ha4t.editor.__main__:app --port 8765
# 或
python -m ha4t.editor [-p PORT] [-w WORKSPACE]

# 构建分发包
uv build

# 发布到 PyPI
uv publish

# 代码格式化（dev 依赖）
black ha4t/
isort ha4t/

# 代码检查
flake8 ha4t/

# 构建文档
cd docs && make html
```

## Code Conventions & Common Patterns

### 命名约定

- **模块级常量/单例**：下划线全大写（`_PLATFORMS`, `_INCLUDE_STACK`, `_ocr`）
- **私有方法/助手**：`_` 前缀（`_resolve_selector`, `_to_abs`, `_find_pos_by_*`, `_get_ocr`, `_ensure_hdc`）
- **Selector 元字段**：`_` 前缀（`_parent`, `_doc`）— `@cost_time` 自动剥离
- **规范字段名**：snake_case（`resourceId`, `className`, `xpath`）
- **驱动内部句柄**：`self._d`（AndroidDriver/IOSDriver/HarmonyDriver 统一）
- **测试类**：`Test<Concept>` PascalCase；方法 `test_<what>_<condition>` snake_case
- **用户可见日志/注释/文档**：简体中文

### 跨平台字段映射核心规则

定义在 `ha4t/selector.py`，无需在其他位置重复：

| 规范字段 | Android | iOS | Harmony |
|---|---|---|---|
| text | text | label | text |
| resourceId | resourceId | name | — |
| label | description | label | text |
| className | className | className | — |
| index | index | (dropped) | — |

### 异常层次

所有异常继承自 `HA4TError(Exception)`：
- `DeviceConnectionError`（also `ConnectionError`）
- `PlatformNotSupportedError`（also `NotImplementedError`，携带 op+platform）
- `ElementNotFoundError`
- `ElementWaitTimeoutError`（also `TimeoutError`）
- `ImageMatchError`
- `OCRTimeoutError`（also `TimeoutError`）
- `AssertionFailedError`（also `AssertionError`）
- `FileTransferError`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Exuils/HA4T](https://github.com/Exuils/HA4T) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
