---
trigger: always_on
description: DeepSearchAgent implements two agent paradigms: "CodeAct Agent" & normal "ReAct Agent". Version 0.2.4.dev introduces comprehensive enhancements to both agent types with a focus on text chunking, prompt organization, and planning capabilities.
---

# Agent Architecture

DeepSearchAgent implements two agent paradigms: "CodeAct Agent" & normal "ReAct Agent". Version 0.2.4.dev introduces comprehensive enhancements to both agent types with a focus on text chunking, prompt organization, and planning capabilities.

## Architecture Overview

The codebase implements a dual-agent architecture pattern:

1. **CodeAct Agent**: Generates and executes Python code to perform research actions
2. **ReAct Agent**: Uses structured JSON for tool calling with explicit reasoning steps

Both agent types share common tools but differ in how they invoke these tools and process the results. The architecture includes a sophisticated template system and state management across both paradigms.

## CodeAct Agent Implementation

[codact_agent.py](mdc:src/agents/codact_agent.py) implements the Code Execution paradigm:

- Based on `smolagents.CodeAgent` - generates executable Python code to perform actions
- Uses `create_codact_agent()` factory function for agent initialization
- Extends the base smolagents prompt templates with custom extensions
- Maintains persistent variables between execution steps for state management
- Implements sophisticated planning at regular intervals (default every 4 steps)
- Streaming support through optional `StreamingCodeAgent` wrapper

### Key Technical Features:

- **Template Merging System**: Uses `merge_prompt_templates()` to combine base templates from smolagents with custom extensions
- **Structured State Management**: Maintains global variables (`visited_urls`, `search_queries`, `key_findings`, etc.)
- **Authorized Imports Management**: Carefully controls which Python modules can be used in the execution environment
- **Tool Integration**: Provides direct access to tools as callable Python functions
- **Failure Handling**: Implements robust error checks and safe access patterns for tools and variables
- **Periodic Planning**: Reassesses strategy at configurable intervals via `planning_interval` parameter

## ReAct Agent Implementation

[agent.py](mdc:src/agents/agent.py) implements the Reasoning + Acting paradigm:

- Based on `smolagents.ToolCallingAgent` - uses structured tool calling via JSON
- Uses `create_react_agent()` factory function for agent initialization
- Uses complete `PromptTemplates` structure for all agent interaction
- Implements explicit reasoning (Thought) before each tool call (Action)
- Supports periodic planning via `planning_interval` parameter (default 7 steps)
- Streaming via optional `StreamingReactAgent` wrapper

### Key Technical Features:

- **JSON-based Tool Calls**: Uses structured JSON format for tool invocation
- **Explicit Reasoning Traces**: Records thought process before each action
- **State Tracking**: Maintains state through step memory rather than variables
- **Tool Integration**: Consistent tool interface with both agent paradigms
- **Standardized Prompt Structure**: Uses comprehensive PromptTemplates system
- **Periodic Planning**: Reassesses strategy at regular intervals via `planning_interval`

## Prompt Template System

The [prompt_templates](mdc:src/agents/prompt_templates) directory contains a modular prompt template system:

- **[codact_prompts.py](mdc:src/agents/prompt_templates/codact_prompts.py)**: Contains:
  - `CODACT_SYSTEM_EXTENSION`: System prompt extension for CodeAct agent
  - `PLANNING_TEMPLATES`: Initial and update planning prompts
  - `FINAL_ANSWER_EXTENSION`: Final answer generation formats
  - `MANAGED_AGENT_TEMPLATES`: Agent team coordination patterns
  - `merge_prompt_templates()`: Function to merge with smolagents base templates

- **[react_prompts.py](mdc:src/agents/prompt_templates/react_prompts.py)**: Contains:
  - Complete `REACT_PROMPT` structures using smolagents `PromptTemplates` class
  - System prompts, tool descriptions and examples
  - Workflow explanations and best practices
  - Planning templates for strategic assessment
  - Final answer formatting requirements

- **[__init__.py](mdc:src/agents/prompt_templates/__init__.py)**: Exports the prompt components for both agent types

## Text Chunking System

- **[chunk.py](mdc:src/agents/tools/chunk.py)**: Agent tool interface utilizing Jina AI Segmenter
  - Implements `ChunkTextTool` class inheriting from `smolagents.Tool`
  - Provides consistent interface across both agent paradigms
  - Passes chunking requests to the core segmenter implementation

- **[segmenter.py](mdc:src/agents/core/chunk/segmenter.py)**: Core segmentation implementation
  - `JinaAISegmenter` class: Primary implementation using Jina AI Segment API
  - Provides both synchronous and asynchronous interfaces
  - Implements robust error handling and retry mechanisms
  - Includes batch processing capability for multiple texts
  - `Chunker` wrapper class: Maintains backward compatibility with original interface

## Streaming Architecture

### ⚠️ Important Note: Streaming Functionality Not Recommended
- Current streaming implementation has known issues and stability concerns
- **Developers and users are advised not to enable streaming mode** until future optimizations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lwyBZss8924d/DeepSearchAgents](https://github.com/lwyBZss8924d/DeepSearchAgents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
