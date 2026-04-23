---
trigger: always_on
description: > 小爱音箱与外部 AI 服务（小智 AI、OpenClaw）的桥接器。
---

# AGENTS.md

> 小爱音箱与外部 AI 服务（小智 AI、OpenClaw）的桥接器。
> 接管音箱音频输入输出，实现与第三方 AI 的对话。

## 系统架构

详见 [README.md 系统架构](README.md#系统架构)（含 Mermaid 流程图和各模块工作流程）。

## 项目结构

```
open-xiaoai-bridge/
├── main.py                        # 入口：解析环境变量，启动 MainApp
├── config.py                      # 用户配置（唤醒词、路由钩子、TTS、OpenClaw 等）
├── core/
│   ├── app.py                     # MainApp 主控制器（单例，管理生命周期）
│   ├── xiaoai.py                  # XiaoAI 设备接入 / 事件桥接
│   ├── xiaoai_conversation.py     # 小爱连续对话策略
│   ├── xiaozhi.py                 # 小智 AI WebSocket 协议客户端
│   ├── openclaw.py                # OpenClaw 网关客户端（连接、消息、TTS 播放）
│   ├── openclaw_conversation.py   # OpenClaw 连续对话循环（VAD → ASR → Agent → TTS）
│   ├── wakeup_session.py          # 小智唤醒会话状态机
│   ├── ref.py                     # 全局引用注册表（get/set 依赖注入）
│   ├── models/                    # 模型文件（KWS/VAD/ASR，.gitignore 排除）
│   ├── assets/sounds/             # 音效（tts_notify.mp3 等）
│   ├── services/
│   │   ├── speaker.py             # SpeakerManager 音箱硬件控制
│   │   ├── api_server.py          # HTTP REST API（aiohttp）
│   │   ├── audio/
│   │   │   ├── stream.py          # GlobalStream 全局音频流（多路输入广播）
│   │   │   ├── codec.py           # 音频编解码
│   │   │   ├── vad/silero.py      # Silero VAD 语音活动检测（ONNX）
│   │   │   ├── kws/sherpa.py      # Sherpa KWS 关键词唤醒
│   │   │   └── asr/sherpa.py      # Sherpa ASR 离线语音识别（SenseVoice）
│   │   ├── tts/doubao.py          # 豆包 TTS 客户端（火山引擎）
│   │   └── protocols/
│   │       ├── websocket_protocol.py  # 小智 WebSocket 协议实现
│   │       └── typing.py              # 协议类型定义
│   └── utils/
│       ├── logger.py              # 彩色日志（XiaozhiLogger 单例）
│       ├── config.py              # ConfigManager（嵌套路径查询、热重载）
│       ├── config_loader.py       # config.py 动态导入
│       ├── base.py                # 基础工具
│       └── file.py                # 文件工具
├── native/                        # Rust PyO3 扩展（maturin 编译）
│   └── src/
│       ├── lib.rs                 # 模块入口：on_output_data, start_server, stop/start_recording, stop/start_playing
│       ├── server.rs              # WebSocket 音频服务器（TCP :4399）
│       ├── python.rs              # Python 回调注册中心（HashMap）
│       ├── macros.rs              # 辅助宏
│       └── tts/                   # TTS 音频处理（流式、PCM 直通、MP3 解码）
├── app/openclaw/                  # OpenClaw 设备身份存储（Ed25519 密钥）
├── skills/xiaoai-tts/             # Agent 工具：通过 HTTP API 控制小爱播放
└── tests/                         # 测试脚本
```

## 核心组件

### MainApp (core/app.py)

应用主控制器，单例模式，管理全部服务生命周期。

- `instance(enable_xiaozhi, enable_openclaw)` → 单例获取
- `run(enable_api_server)` → 启动各服务
- `set_device_state(state)` → 管理设备状态（IDLE / LISTENING / SPEAKING / CONNECTING）
- `send_text(text)` → 发送文本到小智
- `send_to_openclaw(text, wait_response)` → 发送消息到 OpenClaw（返回 run_id 或回复文本）
- `send_to_openclaw_and_play_reply(text, wait_response)` → 发送并 TTS 播放回复
- `schedule(callback)` → 主线程任务队列
- `shutdown()` → 优雅关闭

**边界约束**:
- `MainApp` 是业务主循环和设备状态的单一入口
- `device_state` 以 `MainApp` 为准，其他模块通过代理回写，不各自维护平行状态
- `MainApp.loop` 是业务协程的主调度循环

### XiaoAI (core/xiaoai.py)

小爱音箱交互接口，类级变量（classmethod 风格）。

- `init_xiaoai()` → 初始化原生服务，注册事件处理
- `on_event(event)` → 处理小爱事件（RecognizeResult / AudioPlayer）
- `on_input_data(data)` / `on_output_data(data)` → 麦克风 / 扬声器音频回调
- `run_shell(script, timeout)` → 远端 shell 执行
- 内部维护独立 `async_loop`（后台线程），仅用于原生扩展回调和事件桥接

**边界约束**:
- 负责设备接入和事件桥接，不承载连续对话策略
- 连续对话状态放在 `xiaoai_conversation.py`
- `async_loop` 不应承载新的业务状态机

### XiaoZhi (core/xiaozhi.py)

小智 AI WebSocket 协议客户端，单例模式。

- `connect()` / `disconnect()` → 连接管理
- `send_audio(frames)` / `send_text(text)` → 发送音频 / 文本
- `send_start_listening(mode)` / `send_abort_speaking(reason)` → 协议命令
- 回调委托：`on_incoming_audio`, `on_incoming_json`, `on_network_error` 等

**边界约束**:
- 只负责协议收发，不负责唤醒策略和连续对话策略
- `session_id` 必须由服务端消息更新，不能长期使用空值发送控制消息

### OpenClawManager (core/openclaw.py)

OpenClaw 网关客户端，管理 WebSocket 连接、消息分发、自动重连、TTS 播放。

- `initialize_from_config(enabled)` → 从 config 初始化
- `connect()` → 建立连接（Ed25519 设备身份认证）
- `send(text, wait_response)` → 发送消息，返回 run_id 或回复文本，失败返回 None
- `send_and_play(text, wait_response)` → 发送并 TTS 播放回复
- `is_connected()` / `is_enabled()` → 状态查询

**内部机制**:
- Ed25519 设备身份认证（密钥存储在 `app/openclaw/identity/`）
- WebSocket ping/pong + tick 事件监控连接健康
- 指数退避重连（初始 1s，最大 60s）
- 请求 ID 映射 `_pending: dict[str, asyncio.Future]` 追踪响应
- TTS 播放：`tts_speaker` 为 `"xiaoai"` 时使用小爱原生 TTS，否则使用豆包 TTS（支持流式）
- Rust TTS 播放使用单一活动 `playback_token`：开始新的 Rust TTS 会使旧 token 失效；`stop_tts_playback(token)` 只应由持有该 token 的调用方定向停止自己的播放

**连接参数限制**:
- `client.id`: 必须是 OpenClaw 预定义常量
- `client.mode`: 必须是预定义常量
- `session_key`: 只从 config.py 读取

### OpenClawConversationController (core/openclaw_conversation.py)

OpenClaw 连续对话控制器。唤醒词触发后进入独立的 VAD → ASR → OpenClaw → TTS 循环。

- `start()` → 进入对话模式
- `stop()` → 退出对话
- `is_active()` → 状态查询

**对话循环** (`_run_one_turn`):
1. VAD 检测语音开始（`_wait_for_speech`）
2. 录制完整语音（VAD 帧 hook）
3. SherpaASR 离线识别
4. 退出关键词检测
5. 发送到 OpenClaw
6. TTS 播放回复（阻塞等待完成）
7. 恢复监听

**回声防护机制**:
- `stop_recording` → kill 远端 arecord → 麦克风物理静音
- TTS 和提示音都在关麦期间播放，开麦后 VAD 从干净状态开始检测
- `VAD.resume()` 会自动 `_reset_state()` + `input_bytes.clear()`，清除旧的 `speech_frames` 和音频流缓冲

**VAD 状态泄漏陷阱**:
- VAD 检测循环持续运行，`speech_frames` 会不断积累音频帧
- 如果 `resume()` 不调用 `_reset_state()`，旧帧（唤醒词回声、TTS 回声）会泄漏到下一轮检测，导致 ASR 识别出幽灵音频
- `pause()` 会调 `_reset_state()`，但 `resume()` 必须也调——两者都需要清理状态

**边界约束**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coderzc/open-xiaoai-bridge](https://github.com/coderzc/open-xiaoai-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
