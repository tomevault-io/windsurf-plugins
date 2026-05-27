---
trigger: always_on
description: Guide to building AI features with LLM API's
---

# Using Openrouter.ai as an LLM API

When building AI apps, my preference is to use a platform called OpenRouter which is a unified interface for LLMs. Rather than directly integrating with openai/anthropic/google etc, needing to get accounts and api keys with each of them, we instead use the openrouter service. Openrouter gives us a single api which uses the same python sdk/client as OpenAI (and other popular LLMs). The reason for giving you this guide is that your training data cutoff does not include the latest OpenAI API specs and instead includes deprecated api information. Use this instead as your guide.

This is README guide with examples for using OpenRouter with the OpenAI Python SDK, including instructions for handling tool calling (previously function calling), and different file types (for multimodality).

# **Quickstart for how to use the OpenAI python client with OpenRouter**

## **Synchronous Usage**

```python
from openai import OpenAI
import os

# gets API Key from my .env file for API_BASE_URL (which we use to override the openai url) and we set the OPENROUTER_API_KEY
client = OpenAI(
    base_url=os.getenv('API_BASE_URL'),
    api_key=os.getenv("OPENROUTER_API_KEY")
)

completion = client.chat.completions.create(
  model="openai/gpt-4o-mini",
  messages=[
    {
      "role": "system",
      "content": "You are a helpful assistant"
    },
    {
      "role": "user",
      "content": "What is the meaning of life?"
    }
  ]
)
print(completion.choices[0].message.content)
```

## **Asynchronous Usage for Parallel API Calls**

When you need to make multiple API calls in parallel (e.g., processing multiple documents), use the async client:

```python
from openai import AsyncOpenAI
import os
import asyncio

async def process_document(client, content):
    """Process a single document with the LLM."""
    completion = await client.chat.completions.create(
        model="openai/gpt-4o-mini",
        messages=[
            {
                "role": "user",
                "content": content
            }
        ]
    )
    return completion.choices[0].message.content

async def process_multiple_documents(documents):
    """Process multiple documents in parallel."""
    client = AsyncOpenAI(
        base_url=os.getenv('API_BASE_URL'),
        api_key=os.getenv("OPENROUTER_API_KEY")
    )
    
    # Create tasks for all documents
    tasks = [process_document(client, doc) for doc in documents]
    
    # Process all documents in parallel
    results = await asyncio.gather(*tasks)
    return results

# Example usage
documents = [
    "Analyze this text...",
    "Summarize this document...",
    "Extract key points from..."
]

# Run the async processing
results = asyncio.run(process_multiple_documents(documents))
```

The async client is particularly useful when:
- Processing multiple documents simultaneously
- Making parallel API calls to improve throughput
- Integrating with other async services or web frameworks
- Building high-performance applications that need to handle multiple requests

## **Handling LLM responses: non-streamed vs streamed**

### **Non-streaming response structure:**

If the `stream` parameter is not set or is set to `false`, the non-streamed completion from the llm comes back in one accumulated response, in this format:

```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gpt-4o-mini",
  "system_fingerprint": "fp_44709d6fcb",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "\n\nHello there, how may I assist you today?",
    },
    "logprobs": null,
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 12,
    "total_tokens": 21
  }
}
```

Handling non-streamed response structure:

```python
completion = client.chat.completions.create(
    model=model,
    messages=[
        {
            "role": "user",
            "content": "Say this is a test",
        },
    ],
)
print(completion.choices[0].message.content)
```

### **Streaming response structure:**

If the `stream` parameter is set to `true` then the streamed completion returns in chunks and looks like this:

```json
{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "system_fingerprint": "fp_44709d6fcb", "choices":[{"index":0,"delta":{"role":"assistant","content":""},"logprobs":null,"finish_reason":null}]}

{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "system_fingerprint": "fp_44709d6fcb", "choices":[{"index":0,"delta":{"content":"Hello"},"logprobs":null,"finish_reason":null}]}
```

Handling streamed response:

```python

stream = client.chat.completions.create(
    model=model,
    messages=[
        {
            "role": "user",
            "content": "How do I output all files in a directory using Python?",
        },
    ],
    stream=True,
    tools=tools,
    tool_choice="auto"
)
for chunk in stream:
    if not chunk.choices:
        continue

    print(chunk.choices[0].delta.content, end="")
print()

```

Importantly, when dealing with streamed response, our code will need to accumulate chunks and parse the chunks for the tool call and arguments.



# **Using Tool Calls with OpenRouter**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisboden/knowledgesync](https://github.com/chrisboden/knowledgesync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
