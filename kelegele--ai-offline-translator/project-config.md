---
trigger: always_on
description: - **语言 / 框架** — Dart + Flutter（v3.9.2 SDK），目标平台 macOS & Android
---

# Repository Guidelines

## 技术栈

- **语言 / 框架** — Dart + Flutter（v3.9.2 SDK），目标平台 macOS & Android
- **推理引擎** — llama.cpp（C++ 静态库，PR #22836 提供 STQ1_0 格式支持）
- **原生桥接** — `translator_engine.hpp`（共享 C++ header）+ ObjC++（macOS）/ JNI（Android）
- **主要依赖** — `file_picker`（模型文件导入）、`url_launcher`（模型下载）、`flutter_lints`（lint 规则）、`cupertino_icons`

## 项目结构与模块组织

| 路径 | 内容 |
| --- | --- |
| `flutter_app/` | Flutter 项目根 — UI（Dart）、测试、平台 runner、原生桥接 |
| `flutter_app/lib/` | Dart 源码 — `features/translator/`（主页面/controller/channel）、`design/`（主题/颜色/间距）、`about/` |
| `flutter_app/native/translator_engine/` | 共享 C++ 推理引擎 — `translator_engine.hpp` + `.cpp` |
| `flutter_app/test/` | Flutter 测试 — widget test + `features/` 下按功能分组的单元测试 |
| `third_party/llama.cpp/` | Git submodule，固定到 PR #22836 commit |
| `scripts/` | 构建与 smoke-test 脚本 — `setup.sh` / `setup.ps1`、`build_android_llama.sh`、`safe_llama_smoketest.py` / `.ps1` |
| `models/` | 模型参考文档 + `.gitignore` 排除的二进制目录（GGUF 文件不入 Git） |
| `docs/` | GitHub Pages 介绍页（`index.html`）、内部设计文档（`internal/`）、计划（`superpowers/`） |
| `DESIGN.md` | 视觉设计系统（MiniMax 风格的颜色、字体、间距、按钮、卡片） |

`docs/` 下的关键文档：

- `docs/models_inventory.md`：本机模型清单与兼容性记录。
- `docs/llama_pr22836_notes.md`：`llama.cpp` PR #22836 与 `STQ1_0` 加载说明。
- `docs/index.html`：GitHub Pages 介绍页（发布页），展示项目特性、架构、下载入口和 Demo 截图，截图素材在 `docs/public/`。
- `docs/flutter_mobile_architecture.md`：Flutter UI 与原生推理层架构。

`third_party/llama.cpp/` 是 Git submodule，固定到 `llama.cpp` `PR #22836` 对应 commit。

## 构建与开发命令

在 `flutter_app/` 下执行（除非另注）：

| 操作 | 命令 |
| --- | --- |
| 安装依赖 | `flutter pub get` |
| 静态分析 | `flutter analyze` |
| 运行测试 | `flutter test` |
| 运行 App | `flutter run -d <device>`（macOS / Android） |
| 构建 APK | `flutter build apk --target-platform android-arm64 --release` |
| 构建 macOS DMG | `flutter build macos --release` |
| Python 脚本 | `uv run scripts/safe_llama_smoketest.py`（禁止裸 `python3`） |
| llama.cpp submodule | `git submodule update --init third_party/llama.cpp` |

所有 Python 操作必须通过 `uv` 执行，例如 `uv run`、`uv add` 或 `uv sync`。即使只使用标准库的内联脚本（如一行的 `uv run python -c "..."`），也必须走 `uv run`，禁止直接调用系统 `python3`。

第三方依赖使用 Git submodule 管理：

- `git submodule update --init third_party/llama.cpp`：初始化 `llama.cpp` submodule。
- `git submodule status`：确认当前 submodule 指针。
- 不要把 `third_party/llama.cpp/` 改回临时 clone；如需更新 PR 指针，应更新 submodule commit 并同步修改 README / setup 脚本中的校验 commit。

### llama.cpp 安全执行规则

- 在 Windows 上，**禁止直接运行交互式** `llama-cli` 做手工验证，优先使用受限脚本：`uv run scripts/safe_llama_smoketest.py`。
- Windows 上默认优先 GPU offload；如果没有明确确认 GPU 可用，禁止运行高负载推理命令。
- 未经明确授权，不要在 Windows 上执行 CPU-only 的长时间模型推理；如必须回退 CPU，只能使用受限参数和超时控制。
- 所有 smoke test 必须满足：非交互、可超时、有限输出、有限线程、有限 `n_ctx`、有限 `n_predict`。
- 新增推理脚本时，必须优先考虑"失败时自动退出、超时 kill、输出上限、资源上限"，避免拖死主机。
- `third_party/llama.cpp` 必须保持支持 `STQ1_0` 的 PR #22836 兼容路径；更新 submodule 前必须先验证 `Hy-MT1.5-1.8B-STQ1_0.gguf` 可加载并能完成最小翻译。

## 模型兼容性与下载源

### 铁律：未在目标设备验证可加载+可翻译的模型，禁止写入 `supported_model_info.dart`

`supported_model_info.dart` 中列出的模型会直接展示给用户下载。如果模型无法加载或推理乱码，用户体验直接崩溃。**任何新增模型必须先在真机上验证：能加载 + 能完成至少一次正确翻译。**

### 当前模型兼容性实测（2026-07-30 Android 真机验证）

| 模型文件 | 加载 | 翻译质量 | 结论 |
| --- | --- | --- | --- |
| `Hy-MT1.5-1.8B-STQ1_0.gguf` | ✅ | 偶发错误 token（STQ 量化精度损失），但可用 | **唯一可用模型** |
| `Hy-MT1.5-1.8B-1.25bit.gguf`（非 STQ） | ❌ 加载失败 | — | 引擎不支持其量化类型 |
| `Hy-MT2-1.8B-1.25Bit.gguf` | ✅ | ❌ 全乱码（如 Hello→中文 输出 `ㄇ'`） | STQ 路径与 Hy-MT2 不兼容，禁止使用 |

根因：推理引擎基于 llama.cpp PR #22836，**只支持 STQ1_0 量化格式**。Hy-MT2 虽然也走 STQ 路径（见 `docs/internal/hy_mt2_1_8b_model_notes.md:11`），但其 STQ 变体不被当前引擎正确反量化，导致输出全是乱码 token。非 STQ 的标准量化文件则直接加载失败。

### 下载源：ModelScope

- 模型下载 URL **必须使用 ModelScope**（`modelscope.cn`），国内速度快。禁止使用 `hf-mirror.com` 或其他 HuggingFace 镜像。
- ModelScope resolve URL 格式：`https://modelscope.cn/models/{namespace}/{model}/resolve/master/{filename}`
- **命名空间差异**：Hy-MT2 在 HuggingFace 上是 `tencent/`，在 ModelScope 上是 `AngelSlim/`。修改 URL 时必须确认 ModelScope 上的实际命名空间。
- 下载 URL 出现在 4 处，必须同步修改：`supported_model_info.dart`、`TranslatorChannelHandler.kt`（Android）、`TranslatorChannelHandler.swift`（macOS）、`model_selection_state_test.dart`（测试）。

## 跨平台原生引擎

`translator_engine.hpp` C++ API 在 macOS 与 Android 间保持不变，平台差异全部隔离在桥接层（`translator_bridge.mm` / `translator_jni.cpp`）。修改推理逻辑时只动共享 header/cpp；修改平台特定行为时只动对应 bridge。

## 编码风格与命名约定

Dart 代码遵循标准 Flutter 格式，两个空格缩进。

- **文件命名** — `snake_case.dart`（源文件与测试文件统一）。
- **类命名** — PascalCase。方法、字段、channel 名称使用 lowerCamelCase。
- **测试文件** — `_test.dart` 后缀，与 `lib/features/` 对应放在 `test/features/` 下。
- **Import** — 项目内部引用使用 `package:ai_offline_translator/`。
- **原生桥接文件** — 按职责命名，例如 `translator_engine.cpp`、`TranslatorChannelHandler.kt`。

Markdown 标题应清晰描述内容，仓库内文件链接使用相对路径。

## UI 与视觉规范

涉及 Flutter UI、页面布局、组件样式、颜色、字体、间距或交互状态的改动，必须先阅读并遵循 `DESIGN.md`。

- `DESIGN.md` 是当前项目的视觉规范来源，基于 MiniMax 风格。
- 首版 UI 应保持工具型翻译界面，不做营销页或 landing page。
- 使用 `DESIGN.md` 中的颜色、圆角、间距、按钮、输入框和卡片规则作为默认设计系统。
- 如 Flutter 默认字体或平台字体无法直接使用 `DM Sans`，应先保持排版比例和层级一致，再单独讨论字体引入。

## 测试规范

新增功能应同步补充测试。Flutter 测试放在 `flutter_app/test/`，文件名使用 `_test.dart` 后缀。原生推理代码在接入 UI 前，应提供小型、可重复的测试或脚本，覆盖模型加载、取消推理和错误处理。

当前仓库的本地推理验证统一使用：

- `uv run scripts/safe_llama_smoketest.py`
- 或 `./scripts/safe_llama_smoketest.ps1`

不要把交互式终端会话当成自动化验证方式。

## 提交与 Pull Request 规范

当前提交历史使用简短祈使句，例如 `Add Flutter gitignore`。继续保持这种风格：简洁、现在时、每次提交聚焦一个变更。

Pull Request 应包含简要说明、变更路径、已执行的验证，以及模型或运行时假设。涉及 UI 的 PR 还应附截图或录屏。

## 安全与配置提示


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kelegele/ai-offline-translator](https://github.com/kelegele/ai-offline-translator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
