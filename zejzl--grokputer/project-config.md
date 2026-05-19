---
trigger: always_on
description: **Last Updated**: 2026-01-11
---

# CLAUDE.md - Grokputer Technical Reference for Claude AI

**Last Updated**: 2026-01-11
**Project**: Grokputer - AI-Powered Computer Control & Multi-Agent System
**Status**: Production Ready | Pantheon Operational | MAF Phase 3 Complete | GG Framework Ready

## Overview

Grokputer is a sophisticated multi-agent AI system that combines computer vision, autonomous reasoning, and distributed orchestration. It has evolved from a simple observe-reason-act loop into a production-ready swarm system with self-improvement capabilities.

## Current Architecture Status

### Phase Completion (as of Jan 11, 2026)
- **Phase 0**: Infrastructure & MessageBus - ✅ COMPLETE
- **Phase 1**: Core Agents - ✅ COMPLETE
- **Phase 2**: Pantheon (9 agents) - ✅ COMPLETE (90%)
- **Phase 3**: Multi-Modal Understanding - ✅ COMPLETE
- **Phase 3.5**: MAF (Multi-Agent Framework) - ✅ COMPLETE (100%)
- **Phase 3.6**: GG Framework (Workflow Engine) - 🔄 PLANNING COMPLETE
- **Phase 4**: Self-Improvement & RL - 🔄 STARTING

### Core Systems

#### 1. MessageBus Performance
- **Throughput**: 18,384 msg/sec
- **Latency**: <0.05ms per message
- **Architecture**: Async priority queuing with pub/sub
- **Location**: `src/core/message_bus.py`

#### 2. Pantheon Mode (9-Agent System)
**Agents**:
1. **Observer**: Screen capture, vision processing, OCR
2. **Reasoner**: Task analysis, delegation planning (Coordinator role)
3. **Actor**: Command execution, computer control
4. **Validator**: Safety checks, risk assessment
5. **Learner**: Q-learning with experience replay, adaptive optimization
6. **Memory**: Redis/SQLite persistence, hierarchical storage
7. **Executor**: Multi-step workflow orchestration
8. **Analyzer**: Performance metrics, bottleneck detection
9. **Improver**: Self-healing, proposal application

**Usage**:
```bash
python main.py --pantheon --task "complex multi-agent task"
python main.py -gp --task "Pantheon god mode task"
```

**Architecture Flow**:
```
Learner → Reasoner → Observer → Validator → Executor → Observer → Learner → Analyzer → Improver
```

**Features**:
- Async MessageBus communication between all agents
- Redis persistence for learning state
- Safety validation on all actions
- Performance monitoring and self-improvement
- Docker production deployment ready

#### 3. Multi-Agent Framework (MAF)
**Purpose**: Multi-provider AI collaboration (2-6 providers simultaneously)

**Supported Providers**:
- xAI Grok (grok-4-fast-reasoning, grok-3)
- Anthropic Claude (claude-3-opus, claude-3-sonnet)
- OpenAI (gpt-4, gpt-3.5-turbo)
- Google Gemini (gemini-pro)
- Ollama (local models)

**Features**:
- Weighted voting consensus (configurable weights)
- Role assignment (leader, validator, executor)
- **Phase 3 Complete**: Circuit breakers, retry logic, comprehensive error handling
- Health monitoring with automatic recovery
- Async orchestration with <5s consensus response time
- Provider fallback and graceful degradation
- Performance monitoring and structured logging
- Self-healing capabilities with exponential backoff

**Usage**:
```bash
python main.py --providers grok,claude,openai --task "multi-provider analysis"
python main.py -mb --task "messagebus collab task"
```

**Configuration**: `src/collaboration/configs/`

#### 4. Multi-Modal Understanding
**Capabilities**:
- **Vision**: OCR, color analysis, scene classification, object detection
- **Audio**: STT, MFCC features, emotion inference, voice activity detection
- **Text**: NLP, sentiment analysis, entity extraction
- **Cross-Modal**: Text-vision-audio correlations with confidence scoring

**Components**:
- `src/vision_processor.py` - Image analysis
- `src/audio_processor.py` - Audio processing
- `src/multimodal_processor.py` - Unified processing
- `src/knowledge_graph.py` - Multi-modal knowledge extraction

#### 5. Distributed Swarm
**Features**:
- Docker Swarm with Redis pub/sub
- Multi-replica distribution (3x speed improvement)
- Autonomous daemon with proposal engine
- Prometheus metrics (port 9101)
- Grafana dashboards (port 3000)
- Streamlit monitoring (port 8501)

**Performance**:
- +160% scanner effectiveness post-evolution
- Zero data loss in failover tests
- Auto-apply safe/medium risk fixes

**Usage**:
```bash
python autonomous.py daemon src --auto-propose --replicas 3 --analytics
```

#### 6. GG Framework (Workflow Engine)
**Purpose**: Visual workflow automation system (n8n/Make.com style) for Grokputer

**Status**: 🔄 PLANNING COMPLETE - Ready for implementation

**Components**:
- **BaseNode**: Abstract workflow node class
- **Workflow Engine**: State machine execution engine
- **Flow DSL**: Python-based workflow definition
- **Node Types**: HTTP, Transform, Conditional, AI, Notion, Asana, Slack
- **Pantheon Integration**: MessageBus adapter for agent delegation
- **Self-Healing**: Automatic retry and error recovery

**Architecture**:
```
src/workflow/
├── nodes/
│   ├── base.py          # BaseNode abstract class
│   ├── http.py          # HTTP requests
│   ├── transform.py     # Data transformation
│   ├── conditional.py   # Branching logic
│   ├── ai_node.py       # LLM integration
│   ├── notion.py        # Notion API
│   ├── asana.py         # Asana API
│   └── slack.py         # Slack API
├── engine.py            # Workflow execution engine
├── state.py             # State management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zejzl/grokputer](https://github.com/zejzl/grokputer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
