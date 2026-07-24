---
trigger: always_on
description: This document captures the complete development journey, architecture, and implementation details for the Zep-LiveKit integration project.
---

# Claude's Guide to Zep-LiveKit Integration Development

This document captures the complete development journey, architecture, and implementation details for the Zep-LiveKit integration project.

## Project Overview

**Goal**: Create a comprehensive Zep memory integration for LiveKit agents that provides persistent memory capabilities for voice AI applications.

**Repository**: `integrations/livekit/python/`

**Key Achievement**: Successfully built a production-ready, dual-architecture memory system that provides both conversational memory and knowledge graph capabilities for LiveKit voice agents.

## What We Built

### 1. Dual Agent Architecture

**Two Specialized Agent Classes:**

- **`ZepUserAgent`** (`agent.py`): Thread-based conversational memory
  - Extends LiveKit's `Agent` class
  - Stores conversations in Zep threads using `thread.add_messages()`
  - Retrieves context using `thread.get_user_context()`
  - Perfect for personal assistant scenarios with conversation history
  - Supports context modes: "basic" or "summary"
  - Optional message naming for user and assistant attribution

- **`ZepGraphAgent`** (`agent.py`): Knowledge graph-based memory
  - Extends LiveKit's `Agent` class
  - Stores information in Zep knowledge graphs using `graph.add()`
  - Performs hybrid search across facts, entities, and episodes
  - Uses `compose_context_string()` for smart context composition
  - Perfect for shared knowledge scenarios across multiple users
  - Optional user name prefixing for message attribution


### 2. Key Features Implemented

**Event-Driven Architecture:**
- Uses LiveKit's `conversation_item_added` events for real-time capture
- Automatic conversation capture without manual intervention
- Message deduplication using content hashing and message IDs
- Proper role-based message categorization (user/assistant)

**Memory Storage:**
- Thread-based conversation history storage in `ZepUserAgent`
- Knowledge graph storage in `ZepGraphAgent` with user attribution
- Message attribution with optional user/assistant names
- Error handling with graceful degradation

**Memory Retrieval:**
- Context-aware memory injection in `on_user_turn_completed`
- Thread context retrieval for conversational memory
- Parallel graph search (edges, nodes, episodes) for knowledge memory
- Smart context composition using Zep's utility functions

**LiveKit Integration:**
- Full compatibility with LiveKit Agent ecosystem
- Support for all Agent parameters (STT, LLM, TTS, VAD, tools, etc.)
- Dynamic constructor with `**kwargs: Any` for future-proofing
- Drop-in replacement for standard LiveKit agents

## Development Journey & Problem Solving

### Initial Challenge: Memory Integration Pattern
- **Problem**: How to integrate persistent memory with LiveKit's real-time voice framework
- **Solution**: Event-driven architecture using LiveKit's conversation events
- **Result**: Seamless integration that captures conversations automatically

### Architecture Evolution
- **First Approach**: Single agent class with mixed responsibilities
- **Issue**: Complex codebase with unclear separation of concerns
- **Final Solution**: Dual agent architecture + standalone tools
- **Benefits**: Clear separation, flexible usage patterns, maintainable code

### Message Attribution Requirements
- **Need**: Better tracking of who said what in conversations
- **Implementation**: Optional message naming parameters
- **Result**: `user_message_name` and `assistant_message_name` parameters in `ZepUserAgent`

### Multi-User Considerations
- **Research**: Investigated LiveKit's multi-user capabilities
- **Finding**: Agents are typically instantiated per-user, not as shared instances
- **Solution**: Simple user name prefixing in `ZepGraphAgent` for attribution
- **Deployment Pattern**: Per-user agent instances in production environments


## Current Implementation Status

### ✅ Completed Features
1. **Dual agent architecture** - Thread-based and graph-based memory
2. **Event-driven conversation capture** - Real-time message storage
3. **Memory context injection** - Automatic context retrieval and injection
4. **LiveKit compatibility** - Full Agent ecosystem integration
5. **Message deduplication** - Prevents duplicate storage
6. **Error handling & logging** - Production-ready reliability
7. **Type safety** - Full typing support with proper inheritance
8. **Message attribution** - Optional naming for better conversation tracking
9. **Clean architecture** - Separation between storage and retrieval concerns

### 🏗️ Architecture Patterns

**Thread Memory Pattern (ZepUserAgent):**
```python
# Storage
zep_message = Message(content=user_text.strip(), role="user", name=self._user_message_name)
await self._zep_client.thread.add_messages(thread_id=self._thread_id, messages=[zep_message])

# Retrieval
memory_result = await self._zep_client.thread.get_user_context(
    thread_id=self._thread_id, mode=self._context_mode
)
```

**Knowledge Graph Pattern (ZepGraphAgent):**
```python
# Storage with user attribution
if self._user_name:
    message_data = f"[{self._user_name}]: {user_text}"
await self._zep_client.graph.add(graph_id=self._graph_id, type="message", data=message_data)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getzep/zep](https://github.com/getzep/zep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
