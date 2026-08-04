---
trigger: always_on
description: This example demonstrates PSTN operations with `send_dtmf` for IVR navigation and `transfer_call` for external call transfers. The agent can navigate automated phone menus and transfer calls to external phone numbers with built-in validation.
---

# Line SDK - Transfer Phone Call Example

## About This Example

This example demonstrates PSTN operations with `send_dtmf` for IVR navigation and `transfer_call` for external call transfers. The agent can navigate automated phone menus and transfer calls to external phone numbers with built-in validation.

> Line is Cartesia's open-source SDK for building real-time voice AI agents that connect any LLM to Cartesia's low-latency text-to-speech, enabling natural conversational experiences over phone calls and other voice interfaces.

## Built-in PSTN Tools

```python
from line.llm_agent import LlmAgent, LlmConfig, end_call, send_dtmf, transfer_call

agent = LlmAgent(
    model="anthropic/claude-haiku-4-5-20251001",
    api_key=os.getenv("ANTHROPIC_API_KEY"),
    tools=[send_dtmf, transfer_call],
    config=LlmConfig(
        system_prompt=SYSTEM_PROMPT,
        introduction=INTRODUCTION,
    ),
)
```

## send_dtmf Tool

Sends DTMF (Dual-Tone Multi-Frequency) tones for IVR menu navigation.

**Accepted values:** `0-9`, `*`, `#`

**Use case:** Navigating automated phone menus ("Press 1 for sales, press 2 for support")

**Yields:** `AgentSendDtmf` event

## transfer_call Tool

Transfers the call to an external phone number.

**Requirements:**
- Phone number must be in E.164 format (e.g., `+14155551234`)
- Built-in validation using `phonenumbers` library
- Optional fixed message is configured on the tool at construction (`transfer_call(message=...)`), not by the LLM at call time

**Destination modes:**

- **Dynamic** (default): the LLM supplies `target_phone_number` at call time (used in this example, where the caller chooses where to connect).
- **Pinned**: set the destination at construction with `transfer_call(target_phone_number="+14155551234")`. The number is validated once at construction and hidden from the LLM, which then only decides *whether* to transfer. Use this for fixed escalation lines (e.g. "transfer to a human/supervisor").

```python
# Escalate to a fixed support line — the LLM never has to supply the number.
transfer_to_human = transfer_call(
    target_phone_number="+14155551234",
    message="Connecting you to a member of our team now.",
)
```

**Yields:** `AgentTransferCall` event

## System Prompt Guidance

Include explicit guidance for PSTN operations in your system prompt:

```python
SYSTEM_PROMPT = """You are a helpful phone assistant that can navigate automated phone systems \
and transfer calls.

You have two special capabilities:
1. **DTMF tones**: When you hear an automated menu asking to "press 1 for sales, press 2 for \
support", use the send_dtmf tool to press the appropriate button.
2. **Call transfer**: When the user wants to be connected to a specific phone number, use the \
transfer_call tool.

When navigating phone menus:
- Listen carefully to the menu options
- Ask the user which option they want if unclear
- Press the appropriate button using send_dtmf

When transferring calls:
- ALWAYS read back the full phone number and ask the user to confirm before transferring
- Only call the transfer_call tool AFTER the user confirms the number is correct
- Phone numbers must be in E.164 format (e.g., +14155551234)
- Example: "I have the number plus 1 4 1 5 5 5 5 1 2 3 4. Is that correct?"

Always be helpful and let the user know what you're doing."""
```

## LlmAgent Configuration

```python
import os
from line.llm_agent import LlmAgent, LlmConfig

agent = LlmAgent(
    model="anthropic/claude-haiku-4-5-20251001",  # LiteLLM format
    api_key=os.getenv("ANTHROPIC_API_KEY"),  # Must be explicitly provided
    tools=[...],
    config=LlmConfig(...),
    max_tool_iterations=10,
)
```

**LlmConfig options:**

- `system_prompt`, `introduction` - Agent behavior
- `temperature`, `max_tokens`, `top_p`, `stop`, `seed` - Sampling
- `presence_penalty`, `frequency_penalty` - Penalties
- `num_retries`, `fallbacks`, `timeout` - Resilience
- `extra` - Provider-specific pass-through (dict)

**Dynamic configuration via Calls API:**

The [Calls API](https://docs.cartesia.ai/line/integrations/calls-api) connects client-side audio (web/mobile apps or telephony) to your agent via WebSocket. When initiating a call, clients can pass agent configuration that your agent receives in `CallRequest`.

Use `LlmConfig.from_call_request()` to allow callers to customize agent behavior at runtime:

```python
async def get_agent(env: AgentEnv, call_request: CallRequest):
    return LlmAgent(
        model="anthropic/claude-haiku-4-5-20251001",
        api_key=os.getenv("ANTHROPIC_API_KEY"),
        tools=[end_call, web_search],
        config=LlmConfig.from_call_request(
            call_request,
            fallback_system_prompt=SYSTEM_PROMPT,
            fallback_introduction=INTRODUCTION,
        ),
    )
```

**How it works:**

- Callers can pass `system_prompt` and `introduction` when initiating a call
- Priority: Caller's value > your fallback > SDK default
- For `system_prompt`: empty string is treated as unset (uses fallback)
- For `introduction`: empty string IS preserved (agent waits for user to speak first)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cartesia-ai/line](https://github.com/cartesia-ai/line) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
