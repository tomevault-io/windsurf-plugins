---
trigger: always_on
description: An Alice Agent is a fundamental component, designed to interact with users and perform various tasks. This document explains the key features and capabilities of an Alice Agent.
---

# Alice Agent

An Alice Agent is a fundamental component, designed to interact with users and perform various tasks. This document explains the key features and capabilities of an Alice Agent.
```User_prompt 
A representation of an AI agent
```
![Agent Represents An AI Agent](../../img/random/ai_agent_represented_small.png)
```Agent_prompt
A futuristic representation of an AI agent. The image shows a sleek, translucent humanoid figure with circuitry and data streams visible within its form. The figure is positioned in a dynamic pose, reaching out with one hand. Its head is slightly oversized, featuring a network of glowing nodes representing neural connections. The background is a abstract digital landscape with flowing lines of code and pulsing energy. The color palette is dominated by cool blues and purples with accents of bright white and electric blue to represent energy and intelligence. The overall style is a blend of digital art and sci-fi concept illustration, creating a sense of advanced technology and artificial consciousness.
```

## Core Components

The AliceAgent is a configurable AI assistant that serves as a bridge between your applications and various AI models and capabilities. At its core, an AliceAgent maintains:

1. **Model Configuration**
   - A dictionary of models for different capabilities (chat, vision, speech, etc.)
   - Each model type (CHAT, VISION, STT, TTS, EMBEDDINGS, IMG_GEN) can have one assigned model
   - At least one model (either CHAT or INSTRUCT) is required for basic functionality

2. **System Message**
   - Defines the agent's base behavior and role
   - Can include special instructions for tool usage and code execution
   - Dynamically updated based on agent permissions and capabilities

3. **Permission Settings**
   - Tool Permissions: Controls the agent's ability to use external functions
     - DISABLED: No tool usage
     - NORMAL: Unrestricted tool usage
     - WITH_PERMISSION: Requires user confirmation
     - DRY_RUN: Simulates tool calls without execution
   
   - Code Execution Permissions: Controls the agent's ability to run code
     - DISABLED: No code execution
     - NORMAL: Executes all code blocks
     - WITH_PERMISSION: Requires user confirmation
     - TAGGED_ONLY: Only executes specially marked code blocks

```python
# Example of a fully configured agent
agent = AliceAgent(
    name="multipurpose_assistant",
    # Model configuration for various capabilities
    models={
        ModelType.CHAT: chat_model,      # For conversation
        ModelType.VISION: vision_model,   # For image analysis
        ModelType.TTS: tts_model,        # For speech generation
        ModelType.STT: stt_model,        # For speech recognition
        ModelType.EMBEDDINGS: emb_model,  # For text embeddings
        ModelType.IMG_GEN: img_model     # For image generation
    },
    # Base behavior definition
    system_message=Prompt(
        content="You are a versatile assistant capable of various tasks..."
    ),
    # Permission configuration
    has_tools=ToolPermission.NORMAL,
    has_code_exec=CodePermission.TAGGED_ONLY,
    max_consecutive_auto_reply=1
)
```

The agent processes all interactions through a message-based architecture, where each interaction (whether it's a simple text response, a tool call, or code execution) is wrapped in a MessageDict structure. This ensures consistent handling and storage of all operations, and allows the agent to:

- Generate text responses using language models
- Analyze and generate images
- Process and generate speech
- Execute code in various programming languages
- Use tools to access external functionality
- Generate text embeddings for semantic operations

When used in applications, agents can be employed in two main contexts:
1. **Chat Environments**: Direct interaction with users, where permissions directly control available capabilities
2. **Task Workflows**: As components of larger processes, where their permissions determine which task nodes can be executed

```python
# Minimal agent for safe chat environments
chat_agent = AliceAgent(
    name="safe_chat_assistant",
    models={ModelType.CHAT: chat_model},
    system_message=Prompt(content="You are a helpful chat assistant"),
    has_tools=ToolPermission.DISABLED,
    has_code_exec=CodePermission.DISABLED
)

# Task-oriented agent with full capabilities
task_agent = AliceAgent(
    name="workflow_assistant",
    models={
        ModelType.CHAT: chat_model,
        ModelType.EMBEDDINGS: emb_model
    },
    system_message=Prompt(content="You are a workflow automation assistant"),
    has_tools=ToolPermission.NORMAL,
    has_code_exec=CodePermission.NORMAL
)
```

## Core Capabilities

### Message Generation
At its core, an AliceAgent can generate responses using various language models. This includes:
- Text generation for conversations and tasks
- Vision responses for image analysis
- Speech transcription and generation
- Image generation from text descriptions
- Text embedding generation

```python
# Basic agent setup for chat
chat_agent = AliceAgent(
    name="chat_assistant",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FiboLLM/FiboLLM](https://github.com/FiboLLM/FiboLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
