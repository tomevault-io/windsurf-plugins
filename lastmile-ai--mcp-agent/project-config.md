---
trigger: always_on
description: The Reliable Conversation Manager (RCM) is a production-ready mcp-agent application that implements research findings from "LLMs Get Lost in Multi-Turn Conversation" to create more reliable multi-turn conversational AI systems. This document describes the current implementation status, architecture, and planned enhancements.
---

# Reliable Conversation Manager (RCM) - Implementation Status & Architecture

## Executive Summary

The Reliable Conversation Manager (RCM) is a production-ready mcp-agent application that implements research findings from "LLMs Get Lost in Multi-Turn Conversation" to create more reliable multi-turn conversational AI systems. This document describes the current implementation status, architecture, and planned enhancements.

### Core Design Principles

1. **Conversation-as-Workflow**: The entire conversation is a single workflow instance, NOT individual turns
2. **Quality-First**: Every response undergoes mandatory quality evaluation and potential refinement
3. **Fail-Fast**: Detect quality issues early and fix them before they compound
4. **Observable**: Every decision point is logged and traceable
5. **Testable**: Components are isolated with clear interfaces

## Architecture Decisions

### Why mcp-agent?

The mcp-agent framework provides critical abstractions that align perfectly with RCM requirements:

```python
# From examples/basic/mcp_basic_agent/main.py - canonical agent pattern
async with finder_agent:
    logger.info("finder: Connected to server, calling list_tools...")
    result = await finder_agent.list_tools()
    llm = await finder_agent.attach_llm(OpenAIAugmentedLLM)
```

**Decision**: Use mcp-agent's Agent abstraction for ALL LLM interactions, including quality evaluation. This ensures consistent tool access, logging, and error handling.

### Workflow Architecture Pattern

Based on analysis of mcp-agent examples, there are two patterns:

1. **Turn-as-Workflow** (REJECTED):

```python
# From original design doc - this neutralizes Temporal benefits
@app.workflow
class TurnProcessorWorkflow(Workflow[Dict[str, Any]]):
    async def run(self, args: Dict[str, Any]) -> WorkflowResult[Dict[str, Any]]:
        # Process one turn... loses conversation state
```

2. **Conversation-as-Workflow** (ADOPTED):

```python
# From examples/mcp_agent_server/temporal/basic_agent_server.py - pattern we'll extend
@app.workflow
class BasicAgentWorkflow(Workflow[str]):
    @app.workflow_run
    async def run(self, input: str = "What is the Model Context Protocol?") -> WorkflowResult[str]:
        # Maintains state across entire conversation
```

**Decision**: Implement conversation-as-workflow with internal state management and user input waiting.

### Quality Control Architecture

The paper identifies four key failure modes:

1. **Premature Answer Attempts** (39% of failures)
2. **Answer Bloat** (20-300% length increase)
3. **Lost-in-Middle-Turns** (forget middle context)
4. **Unreliability** (112% increase in multi-turn)

**Decision**: Implement mandatory quality pipeline with LLM-as-judge pattern:

```python
# Based on paper's quality dimensions
quality_dimensions = {
    "clarity": "Clear, well-structured response",
    "completeness": "Addresses all user requirements",
    "assumptions": "Minimizes unsupported assumptions (LOWER IS BETTER)",
    "verbosity": "Concise without bloat (LOWER IS BETTER)",
    "premature_attempt": "Boolean - attempted answer without info",
    "middle_turn_reference": "References information from middle turns",
    "requirement_tracking": "Tracks user requirements across turns"
}
```

## Implementation Status

### ✅ **FULLY IMPLEMENTED (Production Ready)**

- **Complete Quality Control Pipeline**: 7-dimension LLM evaluation with refinement loops working in production
- **Research-Based Data Models**: All conversation models with state persistence and serialization
- **AsyncIO Workflow**: Production REPL with rich formatting and real-time progress reporting
- **Requirement Tracking**: Cross-turn requirement extraction and status management
- **Context Consolidation**: Prevents lost-in-middle-turns (every 3 turns by default)
- **Robust Fallback System**: Comprehensive heuristic fallbacks when LLM providers unavailable
- **Comprehensive Testing**: Automated 3-turn conversation tests with detailed validation
- **Research Metrics**: Answer bloat tracking, premature attempt detection, quality trend analysis
- **Rich REPL Interface**: Interactive commands (/stats, /requirements, /config, /exit) with enhanced formatting
- **Real LLM Integration**: Works with OpenAI and Anthropic APIs via mcp-agent patterns

### 🔄 **PLANNED ENHANCEMENTS**

- **Temporal Workflow Support**: Long-running conversation support (Phase 6 planned)
- **Specialized Task Handlers**: Code vs chat distinction with Claude Code SDK integration
- **Advanced MCP Patterns**: Sophisticated tool selection and usage patterns

## Current Architecture

### File Structure
```
examples/reliable_conversation/
├── src/
│   ├── workflows/
│   │   └── conversation_workflow.py    # Main AsyncIO workflow (Temporal ready)
│   ├── models/
│   │   └── conversation_models.py      # Research-based data models
│   ├── tasks/
│   │   ├── task_functions.py           # Core quality control orchestration
│   │   ├── llm_evaluators.py          # LLM evaluation with fallbacks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lastmile-ai/mcp-agent](https://github.com/lastmile-ai/mcp-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
