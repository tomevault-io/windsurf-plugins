---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

本文件为 Claude Code (claude.ai/code) 提供项目指导。

## 项目概述

Yanfeng AI Task 是基于 ModelScope API 的 Home Assistant AI 集成，采用 **Subentry-Only 架构**，现已整合**三层意图识别能力**，实现快速设备控制和智能对话的完美结合。

### 核心特性
- 🤖 **对话代理** - 支持中文和多语言自然语言对话
- 📝 **AI 任务生成** - 生成文本、结构化 JSON 数据
- 🖼️ **图像生成/识别** - 使用 ModelScope 图像模型
- ⚡ **三层意图识别** - 智能设备控制 + AI 对话
- 🏗️ **Subentry-Only 架构** - 完全基于子配置项的模块化设计

---

## 🏗️ 架构特点：Subentry-Only 设计

### 与传统架构的区别

**传统架构（Main + Subentry）：**
```python
# 主配置项和子配置项都能创建实体
async def async_setup_entry(hass, entry):
    # 为主配置项创建实体
    async_add_entities([MainEntity(entry)])

    # 为子配置项创建实体
    for subentry in entry.subentries.values():
        async_add_entities([SubEntity(entry, subentry)])
```

**本项目架构（Subentry-Only）：**
```python
# 只为子配置项创建实体，主配置项仅用于初始化
async def async_setup_entry(hass, entry, async_add_entities):
    # 只遍历 subentries，不创建主配置项实体
    for subentry in entry.subentries.values():
        if subentry.subentry_type != "conversation":
            continue
        async_add_entities(
            [YanfengAIConversationEntity(entry, subentry)],
            config_subentry_id=subentry.subentry_id,
        )
```

### 设计优势
- ✅ **更模块化** - 每个功能都是独立的子配置项
- ✅ **更灵活** - 用户可以创建多个不同配置的代理
- ✅ **更清晰** - 配置层次结构明确
- ✅ **更易维护** - 避免主/子配置项混合的复杂性

### 实体初始化模式

```python
class YanfengAIConversationEntity(ConversationEntity, YanfengAILLMBaseEntity):
    def __init__(self, entry: ConfigEntry, subentry: ConfigSubentry) -> None:
        # 注意：subentry 是必需参数（不是 Optional）
        super().__init__(entry, subentry)

        # 所有配置都从 subentry.data 读取
        options = self.subentry.data
        llm_api_enabled = self.subentry.data.get(CONF_LLM_HASS_API, False)

        # unique_id 使用 subentry_id
        self._attr_unique_id = subentry.subentry_id
```

---

## 🎯 三层处理机制

借鉴智谱AI集成的设计理念，结合本项目的 Subentry-Only 架构：

### 第一层：意图识别层（Intent Recognition）
- **响应速度**：50-200ms
- **处理方式**：正则表达式 + Intent Handler
- **适用场景**：简单直接的设备控制命令
- **实现文件**：`intents.py` + `intents.yaml`
- **注册位置**：`__init__.py` 中全局注册，所有 subentry 共享

**支持的控制命令：**
```
✅ "打开卧室空调"
✅ "把空调调到26度"
✅ "空调设置制冷模式"
✅ "调高空调风速"
✅ "关闭所有窗帘"
✅ "打开客厅灯"
✅ "通知：明天开会"
```

### 第二层：AI意图理解层
- **响应速度**：500-1500ms
- **处理方式**：AI 解析 + 工具调用
- **适用场景**：复杂的多步骤任务
- **实现文件**：`conversation.py` 中的 AI 处理

### 第三层：AI对话层
- **响应速度**：1-3秒
- **处理方式**：完整的 LLM 对话
- **适用场景**：开放性问答、知识咨询
- **实现文件**：`entity.py` 中的 `_async_handle_chat_log`

---

## 📁 关键文件说明

### 核心模块

#### `__init__.py`
- 集成入口，负责初始化和全局服务注册
- **关键改动**：
  - 在 `async_setup_entry` 中全局注册意图处理器
  - 意图处理器只注册一次，被所有 subentry 共享
  - 没有 `add_update_listener`（与传统架构不同）

```python
async def async_setup_entry(hass: HomeAssistant, entry: YanfengAIConfigEntry) -> bool:
    # 创建 session 并测试连接
    session = aiohttp.ClientSession(...)
    entry.runtime_data = session

    # 全局注册意图处理器（Layer 1 of three-layer processing）
    LOGGER.info("注册意图处理器...")
    await async_setup_intents(hass)
    LOGGER.info("意图处理器注册完成")

    # 设置平台（只为 subentries 创建实体）
    await hass.config_entries.async_forward_entry_setups(entry, PLATFORMS)

    return True
```

#### `conversation.py`
- 对话实体，处理所有用户输入
- **Subentry-Only 适配**：
  - `__init__` 中 `subentry` 参数是必需的（不是 Optional）
  - 直接使用 `self.subentry.data`，无需 None 检查

**三层处理逻辑**：
```python
async def _async_handle_message(self, user_input, chat_log):
    # Layer 1: Intent Recognition
    intent_result = await self._try_intent_recognition(user_input)
    if intent_result:
        LOGGER.info("✅ 第一层成功: 意图识别匹配 - %s", intent_result.intent.intent_type)
        response_text = intent_result.speech.get("plain", {}).get("speech", "")
        if response_text:
            # 添加到对话日志
            assistant_content = conversation.AssistantContent(
                agent_id=self.entry.entry_id,
                content=response_text
            )
            chat_log.content.append(assistant_content)
            return conversation.async_get_result_from_chat_log(user_input, chat_log)

    LOGGER.debug("⚠️ 第一层未匹配，转到第二/三层: AI处理")

    # Layer 2 & 3: AI Processing
    # 从 subentry.data 获取配置（不是 entry.options）
    options = self.subentry.data

    await chat_log.async_provide_llm_data(
        user_input.as_llm_context(DOMAIN),
        options.get(CONF_LLM_HASS_API),
        options.get(CONF_PROMPT),
        user_input.extra_system_prompt,
    )

    await self._async_handle_chat_log(chat_log)
    return conversation.async_get_result_from_chat_log(user_input, chat_log)
```

#### `intents.py`
- 定义所有意图处理器（Intent Handlers）
- **主要意图**：
  - `ClimateSetTemperatureIntent` - 空调温度控制（带智能模式判断）
  - `ClimateSetModeIntent` - 空调模式设置
  - `ClimateSetFanModeIntent` - 风速控制
  - `CoverControlAllIntent` - 批量窗帘控制
  - `HassLightSetAllIntent` - 灯光控制
  - `HassNotifyIntent` - 通知创建

**设计模式**：
```python
class BaseIntent(intent.IntentHandler):
    """Base intent handler."""

    def __init__(self, hass: HomeAssistant) -> None:
        self.hass = hass

class ClimateSetTemperatureIntent(BaseIntent):
    """Handle climate set temperature intent."""

    intent_type = INTENT_CLIMATE_SET_TEMP
    slot_schema = {
        vol.Required("name"): str,
        vol.Required("temperature"): vol.Any(str, int, float)
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yanfeng17/yanfeng_ai_task](https://github.com/yanfeng17/yanfeng_ai_task) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
