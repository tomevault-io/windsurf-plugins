---
trigger: always_on
description: NekroAgent 插件开发规范，包含插件模板、API 用法、SandboxMethodType 选择、提示词注入、错误处理等完整指引。在开发、修改或审查 NekroAgent 插件代码时应用此规则。
---

# NekroAgent 插件开发规则

## 基础模板

```python
from nekro_agent.api import core, message
from nekro_agent.api.plugin import ConfigBase, NekroPlugin, SandboxMethodType
from nekro_agent.api.schemas import AgentCtx
from pydantic import Field

plugin = NekroPlugin(
    name="插件名称",
    module_name="模块名",  # 必须与可导入的模块名一致
    description="插件描述",
    version="0.1.0",
    author="作者",
    url="项目地址",
    support_adapter=["onebot_v11", "minecraft", "sse"],
)

@plugin.mount_config()
class PluginConfig(ConfigBase):
    SETTING: str = Field(default="默认值", title="显示名", description="描述")

config = plugin.get_config(PluginConfig)
store = plugin.store  # 数据存储

@plugin.mount_sandbox_method(SandboxMethodType.TOOL, "方法名")
async def method_name(_ctx: AgentCtx, param: str) -> str:
    """方法描述

    Args:
        param (str): 参数描述

    Returns:
        str: 返回值描述
    """
    # 实现逻辑
    return "结果"

@plugin.mount_cleanup_method()
async def clean_up():
    """清理方法"""
    pass
```

## 核心 API

### 消息发送

```python
# 发送文本
await message.send_text(chat_key, "消息内容", _ctx)

# 发送图片/文件
await message.send_image(chat_key, "图片路径", _ctx)
await message.send_file(chat_key, "文件路径", _ctx)
```

### 数据存储

```python
# 保存数据
await store.set(chat_key="聊天ID", store_key="键", value="值")

# 读取数据
data = await store.get(chat_key="聊天ID", store_key="键")

# 全局存储（不指定chat_key）
await store.set(store_key="全局键", value="值")
```

## SandboxMethodType 详细对比

### 1. TOOL - 工具执行型

**用途：** 直接执行操作，立即生效
**返回值：** 执行结果描述（string）
**AI 处理：** AI 收到结果后继续对话，不会重新分析

```python
@plugin.mount_sandbox_method(SandboxMethodType.TOOL, "发送消息")
async def send_message(_ctx: AgentCtx, chat_key: str, content: str) -> str:
    """发送消息到聊天"""
    await message.send_text(chat_key, content, _ctx)
    return "消息已发送"  # 简单确认信息
```

### 2. AGENT - 分析代理型

**用途：** 获取信息供 AI 分析，不直接执行操作
**返回值：** 详细信息内容（string）
**AI 处理：** AI 会基于返回内容进行分析和决策

```python
@plugin.mount_sandbox_method(SandboxMethodType.AGENT, "搜索信息")
async def search_info(_ctx: AgentCtx, query: str) -> str:
    """搜索相关信息"""
    results = await external_search(query)
    # 返回详细信息供AI分析
    return f"搜索结果：{results}\n请根据以上信息回答用户问题"
```

### 3. BEHAVIOR - 行为状态型

**用途：** 修改状态、记录信息，影响后续对话
**返回值：** 操作确认（bool/string）
**AI 处理：** AI 知道状态已改变，会考虑新状态继续对话

```python
@plugin.mount_sandbox_method(SandboxMethodType.BEHAVIOR, "设置心情")
async def set_mood(_ctx: AgentCtx, chat_key: str, mood: str) -> bool:
    """设置当前心情状态"""
    await store.set(chat_key=chat_key, store_key="mood", value=mood)
    return True  # 状态已改变
```

### 4. MULTIMODAL_AGENT - 多模态代理型

**用途：** 提供多媒体内容供 AI 观察分析
**返回值：** OpenAI 消息格式（Dict）
**AI 处理：** AI 会"看到"图片等多媒体内容并进行分析

```python
@plugin.mount_sandbox_method(SandboxMethodType.MULTIMODAL_AGENT, "显示图片")
async def show_image(_ctx: AgentCtx, image_path: str) -> Dict:
    """显示图片供AI观察"""
    from nekro_agent.services.agent.creator import ContentSegment, OpenAIChatMessage

    msg = OpenAIChatMessage.create_empty("user")
    msg = msg.add(ContentSegment.text_content("这是请求的图片："))
    msg = msg.add(ContentSegment.image_content_from_path(image_path))
    msg = msg.add(ContentSegment.text_content("请描述你看到的内容"))

    return msg.to_dict()  # 返回标准OpenAI消息格式
```

### 类型选择指南

| 类型                 | 使用场景                     | 返回值类型          | AI 后续行为      |
| -------------------- | ---------------------------- | ------------------- | ---------------- |
| **TOOL**             | 发送消息、文件操作、立即执行 | `str` 简短确认      | 继续对话         |
| **AGENT**            | 搜索信息、获取数据、外部查询 | `str` 详细内容      | 分析内容后回复   |
| **BEHAVIOR**         | 状态修改、记录信息、设置配置 | `bool/str` 操作确认 | 考虑新状态继续   |
| **MULTIMODAL_AGENT** | 图片分析、多媒体展示         | `Dict` OpenAI 格式  | 观察多媒体后分析 |

### 实际效果对比

```python
# ❌ 错误用法：用TOOL返回搜索结果
@plugin.mount_sandbox_method(SandboxMethodType.TOOL, "搜索")
async def bad_search(_ctx: AgentCtx, query: str) -> str:
    return "找到了很多结果..."  # AI无法利用具体结果

# ✅ 正确用法：用AGENT返回详细搜索结果
@plugin.mount_sandbox_method(SandboxMethodType.AGENT, "搜索")
async def good_search(_ctx: AgentCtx, query: str) -> str:
    results = await search_api(query)
    return f"搜索'{query}'的结果：\n{results}\n\n请基于以上信息回答"

# ❌ 错误用法：用AGENT执行发送操作
@plugin.mount_sandbox_method(SandboxMethodType.AGENT, "发送消息")
async def bad_send(_ctx: AgentCtx, content: str) -> str:
    await send_message(content)
    return "已发送"  # AGENT不应直接执行操作

# ✅ 正确用法：用TOOL执行发送操作
@plugin.mount_sandbox_method(SandboxMethodType.TOOL, "发送消息")
async def good_send(_ctx: AgentCtx, content: str) -> str:
    await send_message(content)
    return "消息已发送"  # TOOL用于直接执行
```

### 核心设计理念

**执行 vs 观察：**

- **TOOL/BEHAVIOR** = AI **执行操作**，改变外部状态
- **AGENT/MULTIMODAL_AGENT** = AI **获取信息**，用于决策

**返回值用途：**

- **TOOL** 返回值 → 告知 AI 操作结果，让 AI 继续对话
- **AGENT** 返回值 → 提供详细信息，让 AI 分析思考
- **BEHAVIOR** 返回值 → 确认状态改变，影响 AI 后续行为
- **MULTIMODAL_AGENT** 返回值 → 让 AI"看到"多媒体内容

**常见错误：**

1. 用 AGENT 执行操作（应该用 TOOL）
2. 用 TOOL 返回大量信息（应该用 AGENT）
3. MULTIMODAL_AGENT 返回 string 而不是 Dict
4. BEHAVIOR 不更新状态只返回信息（应该用 AGENT）
5. **错误时返回字符串而非 raise 异常**（AI 无法触发修正）
6. **返回自然语言描述而非纯数据**（AI 难以提取路径/ID 等关键信息）
7. **TOOL 返回需要 AI 进一步处理的数据但标记为 BEHAVIOR**（如上传文件应返回路径供 AI 构建 prompt）

## 实用模式

### 1. 缓存避免重复

```python
_CACHE: Dict[str, Any] = {}

async def cached_method(_ctx: AgentCtx, key: str) -> str:
    if key in _CACHE:
        return _CACHE[key]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KroMiose/nekro-agent](https://github.com/KroMiose/nekro-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
