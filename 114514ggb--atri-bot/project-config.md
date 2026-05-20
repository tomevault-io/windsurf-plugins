---
trigger: always_on
description: - **Entry Point**: `main.py` → `atribot/bot_framework.py`（`BotFramework.create()` 工厂方法）。所有服务在 `initialize()` 中**严格按顺序**注册，初始化完成后调用 `TimeTriggerSupervisor.start()` 启动定时循环。
---

﻿````instructions
# ATRI-bot AI Coding Instructions

## Architecture Overview
- **Entry Point**: `main.py` → `atribot/bot_framework.py`（`BotFramework.create()` 工厂方法）。所有服务在 `initialize()` 中**严格按顺序**注册，初始化完成后调用 `TimeTriggerSupervisor.start()` 启动定时循环。
- **初始化顺序**: `config` → `TimeTriggerSupervisor` → `MCP` → `database` → `LLMSupplier` → `SkillsManager` → `SandBox`(可选) → `memorySystem` → `UserSystem` → `ChatManager` → `EmojiCore` → `PermissionsManagement` → `WebSocket/HTTP` → `SendMessage` → `EventTrigger` → `CommandSystem` → `CommandLoader` → `ToolCalls` → `LLMSupervisor` → `GroupChat`
- **依赖注入**: 使用单例 `DIContainer`（`atribot/core/service_container.py`），通过 `container.get("ServiceName")` 获取实例，`container.register(name, obj, cleanup=None)` 注册，`container.register_cleanup(name, handler)` 单独注册清理回调，`container.exists(name)` 检查存在，`container.shutdown()` 按**逆序**执行所有 cleanup。
- **消息流**: `NapCat`（外部QQ） → `WebSocketClient`（单例） → `message_router.main()` → 群聊白名单校验 → `PermissionsManagement.check_access()` → `CommandSystem` 或 `EventTrigger` 或 `LLMCoordinator`。目前**仅处理群消息**，私聊直接 `return`。
- **数据库**: PostgreSQL + `pgvector`（HNSW 1024维，m=16/ef=64）+ `pgroonga` 扩展。全异步，使用 `async with db as db:` 上下文管理器。Schema 定义在 `docker/db/info.sql`，含自定义枚举 `permission_type`、`memory_category`。
- **配置访问**: `atriConfig` 将 JSON 包装为支持点操作的 `ConfigObject`（`assets/config.json`）。路径统一通过 `config.file_path.*` 访问，均为 `Path` 对象。

## 完整服务名称表
| 服务名 | 类型 | Shutdown | 备注 |
|---|---|---|---|
| `log` | `Logger` | — | 容器初始化时自动注册 |
| `config` | `atriConfig` | — | |
| `database` | `AsyncPostgreSQL` | ✅ `close_pool()` | 需 `async with` 使用 |
| `SendMessage` | `QQAPIClient` | — | |
| `LLMSupplier` | `LLMConnectionManager` | ✅ `close()` | |
| `LLMSupervisor` | `LLMCoordinator` | — | |
| `CommandSystem` | `CommandSystem` | — | |
| `memorySystem` | `memorySystem` | — | |
| `SandBox` | `DockerSandbox` | ✅ `stop()` | 初始化可能失败，使用前调用 `container.exists("SandBox")` |
| `SkillsManager` | `SkillsManager` | — | |
| `MCP` | `FuncCall` | ✅ `terminate()` | MCP通过后台队列异步初始化 |
| `TimeTriggerSupervisor` | `TimeTriggerSupervisor` | ✅ `stop()` | |
| `UserSystem` | `UserSystem` | — | |
| `ChatManager` | `ChatManager` | — | 群聊上下文管理 |
| `EmojiCore` | `EmojiCore` | — | 表情系统 |
| `PermissionsManagement` | `PermissionsManagement` | — | async 创建，权限 0-3 四级 |
| `EventTrigger` | `EventTrigger` | — | |
| `WebSocket` | `WebSocketServer` 或 `WebSocketClient` | ✅ `close()` | 由 `connection_type` 决定 |

## 消息类型系统

### ChatMessage 对象
处理函数的**第一个参数固定为** `message_data: ChatMessage`（`atribot/core/type/chat_message_types.py`）：
```python
@dataclass
class ChatMessage:
    user_id: int
    group_id: int | None   # None = 私聊
    message_id: int
    segments: List[MessageSegment]   # 消息段列表
    time: int              # Unix 时间戳
    sender_nickname: str
    
    @property
    def llm_formatted_message(self) -> str  # AI 可读格式化消息
    @property
    def primeval(self) -> dict               # 原始事件数据
```

### MessageSegment 消息段类型
| 类名 | 用途 | 构造 |
|---|---|---|
| `TextSegment` | 纯文本 | `TextSegment(text)` |
| `ImageSegment` | 图片 | `ImageSegment(url)` |
| `AtSegment` | @用户 | `AtSegment(user_id)` |
| `ReplySegment` | 回复消息 | `ReplySegment(msg_id)` |
| `RecordSegment` | 语音 | `RecordSegment(url)` |
| `VideoSegment` | 视频 | `VideoSegment(url)` |
| `FaceSegment` | QQ 表情 | `FaceSegment(face_id)` |
| `ForwardSegment` | 合并转发 | `ForwardSegment(id)` |
| `JsonSegment` | JSON 卡片 | `JsonSegment(json_str)` |

### MessageBuilder（多模态消息构建）
```python
from atribot.core.type.context_types import MessageBuilder

msg = (MessageBuilder()
    .add_text("说明文字")
    .add_image("https://...")
    .add_image_base64(base64_data, "image/png")
    .add_audio(base64_data, "wav")
    .build())   # → Dict[str, Any]
```

## 权限体系
`PermissionsManagement`（`AsyncPermissionsManagement`）四级权限：
- `0`：黑名单（被封禁）
- `1`：普通用户（默认）
- `2`：管理员
- `3`：Root 用户

`authority_level` 字段含义：`0`=无限制，`1`=普通用户可用，`2`=管理员，`3`=Root。

## Key Extension Patterns

### 1. 添加新命令
- 在 `atribot/commands/<category>/` 下创建目录，`CommandLoader` 自动扫描并加载各子目录的 `__init__.py`。
- 处理函数**第一个参数固定为** `message_data: ChatMessage`，通过 `message_data.group_id`、`message_data.user_id` 等属性访问。
- **三种参数装饰器**（顺序：register_command → option/argument/flag → 处理函数）：

```python
from atribot.core.service_container import container
from atribot.core.type.chat_message_types import ChatMessage

cmd_system = container.get("CommandSystem")
send_message = container.get("SendMessage")

@cmd_system.register_command(
    name="cmd",
    description="命令描述",
    authority_level=1,
    aliases=["别名"],
    examples=["/cmd arg --opt value"]
)
# 位置参数（/cmd value）
@cmd_system.argument(name="param", description="...", required=True, type=str, multiple=False)
# 选项参数（--opt value 或 -o value）
@cmd_system.option(name="opt", short="o", long="--opt", description="...", required=False, default=None, type=str)
# 布尔标志（--flag 或 -f，无值）
@cmd_system.flag(name="verbose", short="v", long="--verbose", description="详细输出")
async def handler(message_data: ChatMessage, param: str, opt: str | None, verbose: bool) -> None:
    await send_message.send_group_msg(message_data.group_id, f"Response: {param}")
```

### 2. LLM Function Calling 工具

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [114514ggb/ATRI-bot](https://github.com/114514ggb/ATRI-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
