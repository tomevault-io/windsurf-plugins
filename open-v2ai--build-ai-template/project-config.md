---
trigger: always_on
description: AI 和 Agent 开发规范和 OpenAI API 集成最佳实践
---


# AI 和 Agent 开发规范

## 核心架构

```bash
api/app/
├── agents/          # Agent 实现
│   ├── llm.py      # LLM 基础封装
│   └── dify.py     # Dify 集成
└── utils/           # AI 工具模块
```

## LLM 集成规范

### OpenAI API 配置

位置：[api/app/agents/llm.py](mdc:api/app/agents/llm.py)

```python
from openai import AsyncOpenAI
from typing import AsyncGenerator, Dict, Any, List, Optional
import logging
import asyncio
from datetime import datetime

from ..core.config import settings
from ..core.exceptions import LLMException

logger = logging.getLogger(__name__)

class LLMService:
    """
    LLM 服务类

    功能:
    - OpenAI API 封装
    - 流式响应处理
    - 错误重试机制
    - Token 使用量统计
    """

    def __init__(self):
        """
        初始化 LLM 服务

        配置:
        - API Key 从环境变量获取
        - 设置超时和重试参数
        - 初始化客户端
        """
        self.client = AsyncOpenAI(
            api_key=settings.AGENT_API_KEY,
            base_url=settings.AGENT_BASE_URL,
            timeout=30.0,
            max_retries=3
        )

        # 默认模型配置
        self.default_model = settings.AGENT_MODEL_NAME or "gpt-4o-mini"
        self.default_temperature = 0.7
        self.default_max_tokens = 2000

    async def create_chat_completion(
        self,
        messages: List[Dict[str, str]],
        model: Optional[str] = None,
        stream: bool = False,
        temperature: Optional[float] = None,
        max_tokens: Optional[int] = None,
        **kwargs
    ):
        """
        创建聊天完成请求

        参数:
        - messages: 消息列表
        - model: 模型名称
        - stream: 是否流式输出
        - temperature: 温度参数
        - max_tokens: 最大 token 数

        返回:
        - OpenAI 响应对象

        异常:
        - LLMException: API 调用失败
        """
        try:
            # 使用默认值填充参数
            model = model or self.default_model
            temperature = temperature if temperature is not None else self.default_temperature
            max_tokens = max_tokens or self.default_max_tokens

            logger.info(f"调用 LLM API: model={model}, stream={stream}, messages_count={len(messages)}")

            response = await self.client.chat.completions.create(
                model=model,
                messages=messages,
                stream=stream,
                temperature=temperature,
                max_tokens=max_tokens,
                **kwargs
            )

            return response

        except Exception as e:
            logger.error(f"OpenAI API 调用失败: {e}")
            raise LLMException(f"AI 服务调用失败: {str(e)}")

# 全局 LLM 服务实例
llm_service = LLMService()
```

### 流式响应处理

位置：[api/app/routers/v1/chat.py](mdc:api/app/routers/v1/chat.py)

```python
from fastapi.responses import StreamingResponse
import json
import asyncio
from typing import AsyncGenerator
from sqlalchemy.orm import Session

from ...agents.llm import llm_service
from ...crud.message import create_message, get_all_messages
from ...crud.chat import get_chat_by_id
from ...models.message import Message

async def stream_chat_response(
    messages: List[Dict[str, str]],
    agent_config: Dict[str, Any],
    chat_id: int,
    user_id: int,
    db: Session
) -> AsyncGenerator[str, None]:
    """
    流式聊天响应生成器

    功能:
    - 实时流式输出 AI 响应
    - 统计 Token 使用量
    - 保存消息到数据库
    - 错误处理和重试

    参数:
    - messages: 对话消息列表
    - agent_config: Agent 配置
    - chat_id: 对话 ID
    - user_id: 用户 ID
    - db: 数据库会话

    生成:
    - SSE 格式的流式数据
    """
    full_response = ""
    token_usage = {
        "prompt_tokens": 0,
        "completion_tokens": 0,
        "total_tokens": 0
    }

    try:
        # 构建完整的消息上下文
        system_prompt = agent_config.get("system_prompt", "")
        if system_prompt:
            full_messages = [{"role": "system", "content": system_prompt}] + messages
        else:
            full_messages = messages

        # 获取模型配置
        model_conf = agent_config.get("model_conf", {})
        model = model_conf.get("model", "gpt-4o-mini")
        temperature = model_conf.get("temperature", 0.7)
        max_tokens = model_conf.get("max_tokens", 2000)

        logger.info(f"开始流式响应生成: chat_id={chat_id}, model={model}")

        # 调用 OpenAI 流式 API
        stream = await llm_service.create_chat_completion(
            messages=full_messages,
            model=model,
            stream=True,
            temperature=temperature,
            max_tokens=max_tokens
        )

        # 流式输出处理
        async for chunk in stream:
            if hasattr(chunk, 'choices') and chunk.choices:
                choice = chunk.choices[0]

                # 处理内容增量
                if hasattr(choice, 'delta') and choice.delta.content:
                    content = choice.delta.content
                    full_response += content

                    # 发送内容块
                    yield f"data: {json.dumps({
                        'content': content,
                        'type': 'content',
                        'chat_id': chat_id,
                        'timestamp': datetime.utcnow().isoformat()
                    }, ensure_ascii=False)}\n\n"

                    # 控制输出速度，提升用户体验
                    await asyncio.sleep(0.02)

                # 处理完成信息和 Token 统计
                if hasattr(choice, 'finish_reason') and choice.finish_reason:
                    if hasattr(chunk, 'usage') and chunk.usage:
                        token_usage = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
