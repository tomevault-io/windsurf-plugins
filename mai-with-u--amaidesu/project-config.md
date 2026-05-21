---
trigger: always_on
description: 为在此代码库中工作的 AI 编码代理提供指南。
---

# AGENTS.md

为在此代码库中工作的 AI 编码代理提供指南。

**本文档已重构为渐进式披露格式**：核心规则在此文件，详细指南请查看 `docs/` 目录。

## 快速导航

| 我想... | 查看文档 |
|---------|---------|
| 快速上手项目 | [快速开始](docs/getting-started.md) |
| 了解代码规范 | [开发规范](docs/development-guide.md) |
| 理解架构设计 | [3域架构](docs/architecture/overview.md) |
| 理解事件系统 | [事件系统](docs/architecture/event-system.md) |
| 开发 Provider | [Provider 开发](docs/development/provider-guide.md) |
| 开发 Pipeline | [管道开发](docs/development/pipeline-guide.md) |
| 管理提示词 | [提示词管理](docs/development/prompt-management.md) |
| 编写测试 | [测试指南](docs/development/testing-guide.md) |

## 重构阶段

**✅ 重构已完成（Iteration 3）**：Provider 自我标准化重构已完成，所有 Provider 直接构造 NormalizedMessage。

### 重构状态

- **Input Provider**：8 个全部完成迁移并完成自我标准化重构
  - 删除 InputCoordinator 和 Normalizer 系统
  - Provider 直接构造 NormalizedMessage
  - Pipeline 移至 ProviderManager 层面统一过滤
- **Decision Provider**：3 个已迁移完成
- **Output Provider**：11 个完成迁移（含新增）
- **Service**：1 个完成迁移

### 架构变更

**重构前**：`Provider → RawData → InputCoordinator → NormalizerRegistry → NormalizedMessage`

**重构后**：`Provider → NormalizedMessage → ProviderManager (Pipeline过滤) → input.message.ready 事件`

### 旧插件备份

旧插件代码已归档到 `plugins_backup/` 目录：
- 保留作为历史参考
- 包含完整的迁移对照表
- 请参考 `plugins_backup/MIGRATION_COMPLETE.md`

**不必担心会破坏性变更**，因为重构已经完成，所有功能都已在新架构中正常工作。

## 核心规范

### 必须遵守

- 移动或者重命名文件的时候注意使用 `git mv` 保留历史记录
- 使用中文和用户沟通以及编写文档、注释
- 需要如实汇报自己的工作进度，不得隐瞒问题不报，不得在未经用户允许的情况下降低任务达成标准
- **提交代码前运行测试**：`uv run pytest tests/` 和 `uv run ruff check .`
- **提交代码前进行格式化**: `uv run ruff format .`

### 禁止事项

| 禁止 | 原因 | 替代方案 |
|------|------|----------|
| ❌ 创建新的 Plugin（插件系统已移除） | 架构已重构为 Provider 系统 | 创建 Provider |
| ❌ 使用服务注册机制（已废弃） | 使用 EventBus | EventBus 事件系统 |
| ❌ 硬编码事件名字符串 | 避免拼写错误 | 使用 `CoreEvents` 常量 |
| ❌ 使用空的 except 块 | 隐藏错误 | 记录日志并处理 |
| ❌ 删除失败的测试来"通过" | 自欺欺人 | 修复代码或测试 |
| ❌ 在修复 bug 时进行大规模重构 | 扩大风险范围 | 只修复 bug |
| ❌ 提交未验证的代码 | 可能破坏构建 | 先运行测试和 lint |
| ❌ 类变量中存储可变对象 | 共享状态问题 | 使用 `__init__` 初始化 |

### 架构约束：3域数据流规则

**严格遵守单向数据流：Input Domain → Decision Domain → Output Domain**

| 禁止模式 | 说明 | 详细规则 |
|---------|------|----------|
| ❌ Output Provider 订阅 Input 事件 | 绕过 Decision Domain，破坏分层 | [数据流规则](docs/architecture/data-flow.md) |
| ❌ Decision Provider 订阅 Output 事件 | 创建循环依赖 | 同上 |
| ❌ Input Provider 订阅 Decision/Output 事件 | Input 应只发布，不订阅下游 | 同上 |

## AudioStreamChannel 音频流系统

AudioStreamChannel 是专门的音频数据传输通道，与 EventBus 分离，用于高效传输大量音频数据。

### 与 EventBus 的关系

- **EventBus**: 用于元数据事件（开始/结束/状态通知）
- **AudioStreamChannel**: 用于音频数据流（chunk 数据传输）

### 发布者（TTS Provider）

所有 TTS Provider 在 `_setup_internal()` 中从 `self._dependencies` 获取 AudioStreamChannel：

```python
self.audio_stream_channel = self._dependencies.get("audio_stream_channel")
```

在音频生成时：
1. 调用 `notify_start(AudioMetadata(text=text, sample_rate=...))` 通知开始
2. 循环调用 `publish(AudioChunk(data=chunk_bytes, ...))` 发布音频块
3. 调用 `notify_end(AudioMetadata(...))` 通知结束

### 订阅者（Avatar Provider, Remote Stream Provider）

订阅者在连接/设置阶段注册回调函数：

```python
await audio_channel.subscribe(
    name="provider_name",
    on_audio_start=self._on_audio_start,
    on_audio_chunk=self._on_audio_chunk,
    on_audio_end=self._on_audio_end,
    config=SubscriberConfig(
        queue_size=100,
        backpressure_strategy=BackpressureStrategy.DROP_NEWEST,
    ),
)
```

回调方法负责：
- 接收 AudioChunk
- 重采样到目标采样率（使用 `resample_audio()`）
- 处理音频数据（口型同步、网络传输等）

### 背压策略

- **BLOCK**: 队列满时阻塞等待
- **DROP_NEWEST**: 丢弃新数据（默认，不阻塞 TTS）
- **DROP_OLDEST**: 替换最旧的数据
- **FAIL_FAST**: 队列满时抛出异常

### 依赖注入链路

```python
# main.py
audio_stream_channel = AudioStreamChannel("tts")
await output_provider_manager.setup(..., audio_stream_channel=audio_stream_channel)

# OutputProviderManager 内部
await self.setup_all_providers(event_bus, audio_stream_channel=audio_stream_channel)

# Provider
self.audio_stream_channel = self._dependencies.get("audio_stream_channel")
```

### 共享类型

以下类型被多个 Domain 共享，因此放在 `src/modules/types/` 中避免循环依赖：

| 类型 | 用途 | 定义位置 |
|------|------|---------|
| `EmotionType` | 情感类型枚举 | `src/modules/types/intent.py` |
| `ActionType` | 动作类型枚举 | `src/modules/types/intent.py` |
| `IntentAction` | 意图动作模型 | `src/modules/types/intent.py` |

**为什么这些类型在 Modules 层？**
- 被 Input/Decision/Output 多个 Domain 使用
- 如果放在任何一个 Domain 中，会导致其他 Domain 依赖它
- 放在 Modules 层可以避免循环依赖

**Decision Domain 特定的类型**：
以下类型位于 `src/modules/types/intent.py` 中（共享类型）：
- `Intent`: 完整的决策意图类（Decision Domain 的核心输出）
- `SourceContext`: 输入源上下文

**如何添加新的共享类型？**
1. 评估类型是否真的需要跨多个 Domain 使用
2. 如果是，添加到 `src/modules/types/` 中的合适文件
3. 更新相关 Domain 的导入语句
4. 运行架构测试验证

## 常用命令

### 包管理器

本项目使用 [uv](https://docs.astral.sh/uv/) 作为 Python 包管理器。

```bash
# 同步依赖
uv sync

# 添加新依赖
uv add package-name

# 移除依赖
uv remove package-name
```

### 运行应用

```bash
# 正常运行
uv run python main.py

# 调试模式
uv run python main.py --debug

# 过滤日志（只显示指定模块）
uv run python main.py --filter EdgeTTSProvider SubtitleProvider
```

### Web Dashboard

项目内置 Web 管理界面，启动后端服务后访问 `http://127.0.0.1:60214`。

```bash
# 后端自动启动（默认启用）
uv run python main.py

# 前端开发模式（需单独启动）
cd dashboard
npm install
npm run dev
```

**配置**（`config.toml`）：

```toml
[dashboard]
enabled = true
host = "127.0.0.1"
port = 60214
```

### 测试

```bash
# 运行所有测试

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mai-with-u/Amaidesu](https://github.com/Mai-with-u/Amaidesu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
