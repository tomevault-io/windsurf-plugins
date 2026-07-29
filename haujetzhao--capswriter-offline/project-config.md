---
trigger: always_on
description: - **离线 (Offline)**: 全本地模型 (ASR, 标点, LLM)，保护隐私。
---

# CapsWriter-Offline 开发指南

## 核心设计 (Core Design)
**"快、准、稳、离线"**
- **离线 (Offline)**: 全本地模型 (ASR, 标点, LLM)，保护隐私。
- **C/S 架构**:
    - **Server**: 主进程处理 WebSocket，**独立子进程** (`multiprocessing.Process`) 运行 AI 模型，确保推理（CPU密集）不阻塞网络心跳。
    - **Client**: 轻量启动，负责全局快捷键监听、录音采集、UI 展示。
- **源代码开放**: 入口 [`start_server.py`](start_server.py) / [`start_client.py`](start_client.py) 为冻结入口；核心源码在 [`core/`](core/) 目录，发行版保留为源码供用户修改。
- **配置化**: [`config_client.py`](config_client.py) / [`config_server.py`](config_server.py) 及 `hot*.txt`、[`LLM/*.py`](LLM/) 位于根目录。
- **版本**: v2.5-alpha（2026-04-28）

## 架构细节与流程 (Architecture & Workflows)

### 1. 识别全链路 (Recognition Flow)
- **采集**: Client 监听快捷键（默认 CapsLock 和 X2）。按下就开始收集录音chunk，超过 **0.3s (Threshold)** 不松则触发识别，**实时流式**通过 WebSocket 发送。
- **切片 (Slicing)**: Client 配置 `mic_seg_duration` (60s) 和 `mic_seg_overlap` (4s)。Server 仅基于时间切片，**禁用 VAD** 以保留完整上下文。
- **Server 处理**:
    - **双重结果**: 同时计算 `text` (简单文本拼接, Robust) 和 `text_accu` (基于 Token 时间戳去重, Precision)。
    - **拼接算法**: `text_accu`使用 **Token 时间戳去重** ([`core/server/merger/`](core/server/merger/))，`text` 使用 **模糊文本匹配**。
- **Client 后处理**:
    - **触发**: 用户**松开按键** -> Server 返回 IsFinal 结果。
    - **热词 (RAG)**: 基于 **音素 (Phoneme)** 的两阶段模糊检索，匹配 `hot.txt`（统一中英文热词）。
    - **规则替换**: `hot-rule.txt` 正则替换。
    - **LLM 润色**: 根据角色配置进行智能润色或回答。
    - **上屏**: 模拟键盘输入或 Toast 显示。

### 2. 客户端模式 (Client Modes)
- **听写 (Dictation)**: 默认模式。按住快捷键 -> 发送音频 -> 松开上屏。
- **转录 (Transcription)**: 拖入文件 -> `ffmpeg` 提取音频 -> 发送 Server -> 接收带时间戳结果 -> 生成 `.srt` / `.txt` / `.json`。

### 3. LLM Agent & 智能修正
- **实时监控 (Hot Reload)**: Client 启动 `watchdog` 文件监视器，实时响应 `hot*.txt` 和 `LLM/*.py` 的修改（3秒防抖）。
- **角色系统**: 模块化的 LLM 角色配置，支持多角色切换。
- **角色触发**: 检测识别结果前缀（如"翻译"、"助理"），匹配 [`LLM/`](LLM/) 下定义的角色。
- **Context 组装**（根据角色配置决定是否启用）:
    1.  **潜在热词**: RAG 检索 `hot.txt`（`enable_hotwords`）。
    2.  **选中文字**: 模拟 Ctrl+C 获取的鼠标选中文本（`enable_read_selection`）。
    3.  **对话历史**: 保留上下文历史记录（`enable_history`）。
    4.  **用户指令**: 当前语音输入内容。
- **输出模式**:
    - **typing**: 直接模拟键盘打字输出。
    - **toast**: 在 Toast 弹窗中显示，支持 Markdown 渲染。
- **UI**: 结果流式显示在 **Toast** (Tkinter 无边框置顶窗)，支持 Markdown 渲染。

### 4. 热词系统 (Hotword System)
- **服务器热词**: `hot-server.txt` 用于服务端热词增强。
- **统一文件**: `hot.txt` 统一管理中英文热词（基于音素匹配）。
- **两阶段检索**:
    1.  **FastRAG**: 倒排索引 + Numba JIT 快速粗筛（减少 90% 计算量）。
    2.  **AccuRAG**: 模糊音权重精确匹配（前后鼻音、平翘舌等）。
- **双阈值机制**:
    - `hot_thresh` (0.85): 高阈值用于实际替换。
    - `hot_similar` (0.6): 低阈值用于 LLM 上下文参考。
- **规则替换**: `hot-rule.txt` 支持正则表达式规则替换 (`pattern = replacement`)。

### 5. 历史归档 (Diary)
- **按日期归档**: `年份/月份/日期.md`。
- **音频**: 原始录音存入 `年份/月份/assets/`，Markdown 中自动生成 HTML 音频控件链接。

### 6. UDP 广播与控制
- **UDP 广播**: 识别结果可通过 UDP 广播到局域网（`udp_broadcast=True`）。
- **UDP 控制**: 支持通过 UDP 命令远程控制录音启停（`udp_control=True`）。

## 关键路径 (Key Paths)
- **服务端配置**: [`config_server.py`](config_server.py) — 模型选择、网络、格式化、对齐器。
- **客户端配置**: [`config_client.py`](config_client.py) — 快捷键、音频、热词、LLM、输出、UDP。
- **热词**:
    - [`hot.txt`](hot.txt) - 统一 RAG 音素匹配（中英文）
    - [`hot-rule.txt`](hot-rule.txt) - 规则替换
    - [`hot-server.txt`](hot-server.txt) - 服务端热词
- **LLM角色**: [`LLM/*.py`](LLM/) (根目录, 定义 Role/Prompt/Model)
    - [`default.py`](LLM/default.py) - 默认角色（热词、润色，process=False）
    - [`翻译.py`](LLM/翻译.py) - 翻译角色（ollama/gemma3:12b）
    - [`高级翻译.py`](LLM/高级翻译.py) - 高级翻译（deepseek/deepseek-chat）
    - [`大助理.py`](LLM/大助理.py) - 大助理（zhipu/glm-4.5-air）
    - [`小助理.py`](LLM/小助理.py) - 小助理（lmstudio/local-model）
- **服务端核心**: [`core/server/`](core/server/)
    - [`app.py`](core/server/app.py) - `CapsWriterServer` 门面类
    - [`state.py`](core/server/state.py) - `ServerState` / `WorkerState` 共享状态
    - [`schema.py`](core/server/schema.py) - `Task` / `Result` / `RecognitionSession` 数据结构
    - [`connection/server_manager.py`](core/server/connection/server_manager.py) - `SocketManager` WebSocket 服务端生命周期
    - [`connection/ws_recv.py`](core/server/connection/ws_recv.py) - 音频接收与切片
    - [`connection/ws_send.py`](core/server/connection/ws_send.py) - 识别结果发送
    - [`worker/process_manager.py`](core/server/worker/process_manager.py) - `ProcessManager` 子进程管理
    - [`worker/worker.py`](core/server/worker/worker.py) - `RecognizerWorker` 推理循环
    - [`worker/model_loader.py`](core/server/worker/model_loader.py) - `ModelLoader` 模型加载
    - [`worker/pipeline.py`](core/server/worker/pipeline.py) - `TaskPipeline` 识别流水线
    - [`engines/`](core/server/engines/) - ASR 引擎实现（见下方模型支持）
    - [`merger/`](core/server/merger/) - 文本/Token 合并算法
    - [`formatter/text_formatter.py`](core/server/formatter/text_formatter.py) - `TextFormatter` 后处理
- **客户端核心**: [`core/client/`](core/client/)
    - [`app.py`](core/client/app.py) - `CapsWriterClient` 门面类
    - [`state.py`](core/client/state.py) - `ClientState` 共享状态
    - [`connection/websocket_manager.py`](core/client/connection/websocket_manager.py) - `WebSocketManager`
    - [`audio/`](core/client/audio/) - `AudioStreamManager` / `Recorder` / `FileManager`
    - [`shortcut/`](core/client/shortcut/) - `ShortcutManager`（pynput）快捷键系统
    - [`output/result_processor.py`](core/client/output/result_processor.py) - `ResultProcessor` 后处理核心
    - [`output/text_output.py`](core/client/output/text_output.py) - `TextOutput` 上屏
    - [`hotword/`](core/client/hotword/) - 热词系统（Phoneme RAG + Rule + Rectification）
    - [`llm/`](core/client/llm/) - LLM 子系统（角色加载、上下文、API 调用）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaujetZhao/CapsWriter-Offline](https://github.com/HaujetZhao/CapsWriter-Offline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
