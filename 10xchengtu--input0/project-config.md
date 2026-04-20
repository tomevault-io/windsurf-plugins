---
trigger: always_on
description: macOS 语音输入工具 — 按住快捷键录音，松开后本地 STT 转文字 → LLM 优化 → 自动粘贴到当前输入框。
---

# Input0

macOS 语音输入工具 — 按住快捷键录音，松开后本地 STT 转文字 → LLM 优化 → 自动粘贴到当前输入框。

## Quick Start

```bash
# 安装前端依赖
pnpm install

# 开发模式（Tauri + Vite 热更新）
pnpm tauri dev

# 运行 Rust 后端测试
cd src-tauri && cargo test --lib

# 前端类型检查
pnpm build   # tsc && vite build

# 生产构建
MACOSX_DEPLOYMENT_TARGET=11.0 CMAKE_OSX_DEPLOYMENT_TARGET=11.0 pnpm tauri build --bundles app
```

前置要求：macOS 11+、Rust stable、Node.js 20+、pnpm、cmake (`brew install cmake`)。首次运行需下载 STT 模型（在 Settings 页面触发）。

## Architecture Overview

Tauri v2 桌面应用，双进程架构：

- **Rust 后端** (`src-tauri/src/`): 音频采集 → 格式转换 → STT 转录 → LLM 优化 → 剪贴板粘贴，通过 Pipeline 状态机驱动。
- **React 前端** (`src/`): 两个窗口 — Settings（主窗口，含 Sidebar 多页面）+ Overlay（语音输入浮层）。
- **通信**: Tauri IPC commands (前端→后端) + Events (后端→前端，`pipeline-state` 事件驱动 UI 状态)。

详见 → [docs/architecture.md](docs/architecture.md)

## Directory Structure

```
.github/workflows/            # CI/CD
└── release.yml               # 推送 release 分支 → 双架构构建 + GitHub Release

src-tauri/src/                # Rust 后端
├── pipeline.rs               # 核心：语音处理流水线状态机
├── lib.rs                    # 应用入口、快捷键注册、STT 模型加载
├── errors.rs                 # 统一错误类型 AppError (thiserror)
├── audio/                    # 音频采集(cpal) + 格式转换(rubato)
│   ├── capture.rs            # AudioRecorder - 麦克风录音
│   ├── converter.rs          # stereo→mono, 重采样16kHz, i16→f32
│   └── tests.rs              # 音频转换单元测试
├── stt/                      # STT 转录抽象层
│   ├── mod.rs                # TranscriberBackend trait + ManagedTranscriber
│   ├── whisper_backend.rs    # whisper-rs 后端 (Metal GPU 加速)
│   ├── sensevoice_backend.rs # sherpa-onnx SenseVoice 后端
│   ├── paraformer_backend.rs # sherpa-onnx Paraformer 后端
│   └── moonshine_backend.rs  # sherpa-onnx Moonshine 后端
├── models/                   # STT 模型管理
│   ├── registry.rs           # 模型注册表（元数据、下载URL、语言推荐）
│   └── manager.rs            # 模型下载/存储/校验
├── llm/                      # LLM 文本优化（GPT API）
├── input/                    # 剪贴板操作 + 模拟粘贴(arboard)
├── config/                   # TOML 配置文件读写
├── vocabulary.rs             # 词汇库存储（JSON 持久化、去重、原子写入）
└── commands/                 # Tauri IPC 命令（每个模块一个文件）
    ├── audio.rs              # start/stop/toggle/cancel_pipeline
    ├── config.rs             # get/save/update_config_field
    ├── whisper.rs            # transcribe/init/is_loaded
    ├── llm.rs                # optimize_text/test_api_connection
    ├── models.rs             # list/download/switch/delete/recommend
    ├── vocabulary.rs         # get/add/remove/validate_and_add vocabulary
    ├── input.rs              # paste_text/parse_hotkey
    └── window.rs             # show/hide overlay/settings

src/                          # React 前端
├── App.tsx                   # BrowserRouter: / → Settings, /overlay → Overlay
├── pages/
│   ├── Settings.tsx          # 主窗口（Sidebar 布局）
│   └── Overlay.tsx           # 语音输入浮层（液态玻璃 UI）
├── components/
│   ├── Sidebar.tsx           # 侧边栏导航
│   ├── HomePage.tsx          # 首页
│   ├── HistoryPage.tsx       # 转录历史记录
│   ├── SettingsPage.tsx      # 配置页面
│   ├── WaveformAnimation.tsx # 录音波形动画
│   ├── ProcessingIndicator.tsx # 处理中指示器
│   └── Toast.tsx             # 提示消息
├── stores/                   # Zustand 状态管理
│   ├── recording-store.ts    # 录音状态
│   ├── settings-store.ts     # 用户配置
│   ├── history-store.ts      # 转录历史
│   ├── theme-store.ts        # 主题状态
│   └── update-store.ts       # 应用更新状态
└── hooks/
    └── useTauriEvents.ts     # Tauri 事件监听 Hook

docs/                         # 设计文档与需求记录
```

## Key Conventions

1. **禁止格式化老代码**: 修改文件时只改逻辑代码，不调整老代码的缩进、空格、空行。Git diff 只包含实质变更。
2. **Feature 文档驱动**: 每个新需求先建 `docs/feature-xxx.md`，记录需求分析、技术方案、实现状态，完成后更新本文件索引。
3. **TDD (后端)**: Rust API 实现先写测试再写代码。测试在模块内的 `tests.rs` 或 `#[cfg(test)]` 中。
4. **错误处理**: 统一使用 `AppError` (thiserror)，按模块分类: Config / Audio / Whisper / Llm / Input / Io。禁止 `unwrap()` / `expect()` 在非初始化代码中使用。
5. **前后端通信**: 后端→前端用 Events (`app_handle.emit("pipeline-state", ...)`)，前端→后端用 Tauri IPC commands。
6. **状态管理**: 前端用 Zustand (一个 store per domain)，后端用 `Arc<Mutex<T>>` 通过 Tauri managed state 共享。
7. **STT 架构**: 通过 `TranscriberBackend` trait 抽象多后端，新增 STT 引擎只需实现该 trait。
8. **类型安全**: TypeScript `strict: true`，Rust 默认 warnings。禁止 `as any` / `@ts-ignore` / `@ts-expect-error`。
9. **Commit message 必须英文**: 所有 git commit message（包括 subject 和 body）使用英文书写，遵循 Conventional Commits 风格（`feat:` / `fix:` / `docs:` / `chore:` 等）。`docs` 和 `chore` 类型不会出现在 GitHub Release 页面（由 `cliff.toml` 过滤）。

详见 → [docs/conventions.md](docs/conventions.md)

## Documentation Map

| 文档 | 用途 | 最后校验 |
|------|------|----------|
| [docs/architecture.md](docs/architecture.md) | 系统架构、模块关系、数据流、分层设计 | 2026-04-03 |
| [docs/conventions.md](docs/conventions.md) | 编码规范、命名约定、模式参考 | 2026-04-03 |
| [docs/init-prd.md](docs/init-prd.md) | 项目初始需求文档（PRD） | 2026-03-29 |
| [docs/feature-scaffold.md](docs/feature-scaffold.md) | Tauri v2 + React + TS 脚手架搭建记录 | 2026-03-29 |
| [docs/feature-audio-converter.md](docs/feature-audio-converter.md) | 音频转换模块（TDD 实现） | 2026-03-29 |
| [docs/feature-pipeline.md](docs/feature-pipeline.md) | 语音处理流水线 + 启动流程 | 2026-03-29 |
| [docs/feature-zh-initial-prompt.md](docs/feature-zh-initial-prompt.md) | 中文转录简繁体修复 | 2026-03-29 |
| [docs/feature-model-switching.md](docs/feature-model-switching.md) | STT 模型切换 + 按需下载 + 推荐 | 2026-04-19 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [10xChengTu/input0](https://github.com/10xChengTu/input0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
