---
trigger: always_on
description: **Purpose**: Central guide for working with the haive-agents package
---

# CLAUDE.md - Haive Agents Package Guide

**Purpose**: Central guide for working with the haive-agents package
**Version**: 1.0
**Last Updated**: 2025-01-18

## 🎯 Package Overview

Haive Agents provides **truly dynamic agents** capable of self-modification, self-replication, runtime adaptation, and autonomous coordination. This is not just another agent framework - it's a system for building agents that can **modify their own behavior**, **spawn new agents**, and **coordinate complex workflows** in real-time.

**Key Innovation**: Agents that adapt, evolve, and self-organize based on task requirements, without predefined configuration.

## 📁 Directory Structure

```
haive-agents/
├── src/haive/agents/
│   ├── __init__.py              # Main exports and module docstring
│   │
│   ├── simple/                  # Foundation agents
│   │   ├── agent.py             # SimpleAgent - conversation, structured output
│   │   ├── agent_v2.py          # Enhanced SimpleAgent with state management
│   │   ├── agent_v3.py          # Latest SimpleAgent with hooks & recompilation
│   │   └── __init__.py          # Simple agent exports
│   │
│   ├── react/                   # ReAct pattern agents
│   │   ├── agent.py             # ReactAgent - tools, reasoning, planning
│   │   ├── agent_v2.py          # Enhanced ReactAgent with better loops
│   │   ├── agent_v3.py          # Latest ReactAgent with V3 improvements
│   │   └── __init__.py          # React agent exports
│   │
│   ├── multi/                   # Multi-agent coordination
│   │   ├── multi_agent.py       # Basic MultiAgent coordination
│   │   ├── enhanced_multi_agent_v3.py  # Enhanced with generics
│   │   ├── enhanced_multi_agent_v4.py  # Latest with full async support
│   │   └── __init__.py          # Multi-agent exports
│   │
│   ├── agent/                   # Advanced agent capabilities
│   │   ├── dynamic_supervisor.py       # DynamicSupervisorAgent
│   │   ├── self_modifying_agent.py     # Self-modification capabilities
│   │   ├── self_replicating_agent.py   # Self-replication system
│   │   ├── provider_switching_agent.py # Provider hot-swapping
│   │   └── __init__.py          # Advanced agent exports
│   │
│   ├── memory_reorganized/      # 🧠 COMPREHENSIVE MEMORY SYSTEM
│   │   ├── __init__.py          # Memory system exports & architecture docs
│   │   ├── agents/              # Memory-enhanced agents
│   │   │   ├── simple.py        # SimpleMemoryAgent with classification
│   │   │   ├── react.py         # ReactMemoryAgent with context
│   │   │   ├── multi.py         # MultiMemoryAgent coordination
│   │   │   └── ltm.py           # LongTermMemoryAgent with consolidation
│   │   ├── retrieval/           # Advanced retrieval systems
│   │   │   ├── graph_rag_retriever.py      # Graph RAG with Neo4j traversal
│   │   │   └── enhanced_retriever.py       # Multi-factor retrieval scoring
│   │   ├── coordination/        # Memory system coordination
│   │   │   ├── integrated_memory_system.py # Unified memory coordination
│   │   │   └── multi_agent_coordinator.py  # Multi-agent memory sharing
│   │   ├── knowledge/           # Knowledge graph management
│   │   │   └── kg_generator_agent.py       # Automatic graph construction
│   │   ├── core/                # Core memory functionality
│   │   │   ├── classifier.py    # LLM-based memory classification
│   │   │   └── types.py         # 11 memory type definitions
│   │   └── search/              # Search agents
│   │       ├── quick_search/    # Fast semantic search (<10ms)
│   │       └── pro_search/      # Deep relationship search (100-500ms)
│   │
│   └── base/                    # Base classes and mixins
│       ├── agent.py             # Base Agent class
│       ├── mixins/              # Capability mixins
│       │   ├── self_modification.py   # Self-modification mixin
│       │   ├── replication.py         # Replication mixin
│       │   ├── provider_switching.py  # Provider switching mixin
│       │   └── tool_transfer.py       # Tool sharing mixin
│       └── __init__.py          # Base exports
│
├── examples/                    # Organized examples
│   ├── basic/                   # Getting started
│   ├── advanced/                # Complex patterns
│   ├── self_modification/       # Self-modifying examples
│   └── dynamic_supervision/     # Supervisor examples
│
├── tests/                       # Comprehensive tests (NO MOCKS)
│   ├── simple/                  # SimpleAgent tests
│   ├── react/                   # ReactAgent tests
│   ├── multi/                   # Multi-agent tests
│   ├── agent/                   # Advanced agent tests
│   └── integration/             # End-to-end tests
│
└── project_docs/               # Package documentation
    ├── guides/                  # Usage guides
    ├── architecture/            # System design
    ├── patterns/                # Implementation patterns
    └── examples/                # Code examples
```

## 🚀 Quick Start Commands

```bash
# Install package with dependencies
poetry install

# Run basic examples
poetry run python examples/basic/simple_agent_example.py
poetry run python examples/basic/react_agent_example.py
poetry run python examples/basic/multi_agent_example.py

# Run advanced examples

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pr1m8/haive-agents](https://github.com/pr1m8/haive-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
