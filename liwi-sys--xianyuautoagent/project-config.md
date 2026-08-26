---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

XianyuAutoAgent is an AI-powered customer service automation system for the Xianyu (闲鱼) platform, China's largest second-hand marketplace. It implements a sophisticated multi-agent architecture that provides 24/7 automated customer support with intelligent conversation routing, price negotiation capabilities, and technical support.

## Project Status
**Current Architecture**: Modular Refactored Design (v2.0)
- **Current Entry Point**: `main.py` (now uses refactored architecture)
- **Core Components**: `core/xianyu_live.py`, modular managers, config system
- **Development Status**: Complete migration to modular architecture

## Development Commands

### Environment Setup
```bash
# Install dependencies
pip install -r requirements.txt

# Configure environment (copy from example)
cp .env.example .env
# Edit .env with your API key and cookies
```

### Running the Application
```bash
# Run with current modular architecture
python main.py

# Run with Docker
docker-compose up -d

# Build custom Docker image
docker build -t xianyu-autoagent .

# Run tests
python run_tests.py
pytest tests/ -v --cov=.
```

### Configuration
- **Unified Config Management**: `config/config.json` with environment variable overrides
- **Environment variables in `.env`** (API_KEY, COOKIES_STR, MODEL_BASE_URL, MODEL_NAME)
- **Agent prompts in `prompts/` directory**
- **SQLite database in `data/` directory**
- **Test configuration**: `pytest.ini` with comprehensive test settings

## Architecture Overview

**Current Architecture** (Modular):
- **Clear Separation of Concerns**: Each module has a single responsibility
- **Dependency Injection**: Loose coupling between components
- **Configuration Management**: Centralized configuration system
- **Testing Framework**: Comprehensive unit and integration tests

### Core Components (Refactored)

**Main Entry Point**: `main.py`
- Clean application initialization
- Configuration management integration
- Service composition and dependency injection
- Graceful shutdown handling

**Configuration System**: `config/config_manager.py`
- Centralized configuration management
- Environment variable and file-based configuration
- Type conversion and validation
- Hot-reload capability
- **Model Routing Configuration**: Intelligent model selection based on intent and complexity
- **Message Batching Configuration**: Configurable batching strategies for performance optimization

**Modular Managers** (`managers/` directory):
- `websocket_manager.py`: WebSocket connection lifecycle management
- `message_processor.py`: Message parsing, validation, and routing
- `message_batcher.py`: Message batching and intent analysis for performance optimization
- `token_manager.py`: Token refresh and authentication
- `heartbeat_manager.py`: Connection health monitoring

**Multi-Agent System**: `XianyuAgent.py` (enhanced)
- `ClassifyAgent`: Intent classification using LLM prompt engineering
- `PriceAgent`: Price negotiation with dynamic temperature strategies based on bargaining rounds
- `TechAgent`: Technical support with web search integration (`enable_search: true`)
- `DefaultAgent`: General customer service with length-limited responses
- `IntentRouter`: Three-tier routing system (technical → price → LLM fallback)
- Safety filtering for platform compliance (blocks: 微信, QQ, 支付宝, 银行卡, 线下)
- **Intelligent Model Routing**: All agents support dynamic model selection based on intent and complexity
- **Performance Optimization**: Smart model selection reduces LLM costs by 82.4%

**API Integration**: `XianyuApis.py` (unchanged)
- Cookie-based authentication with automatic renewal
- Secure token generation with MD5 signing
- Item information retrieval with caching
- Comprehensive retry logic with graceful degradation

**Context Management**: `context_manager.py` (unchanged)
- SQLite database with three main tables:
  - `messages`: Conversation history with chat_id-based session isolation
  - `chat_bargain_counts`: Bargaining round tracking per conversation
  - `items`: Cached item information with automatic expiration
- Configurable history limits with automatic cleanup
- Multi-session support with chat-based isolation

**Utilities**: `utils/xianyu_utils.py` (unchanged)
- MessagePack decryption with fallback mechanisms
- MD5-based request signing for API authentication
- UUID-based device ID generation with user-specific suffixes
- Robust cookie parsing and validation

### Key Design Patterns (Refactored)

1. **Modular Architecture**: Clear separation of concerns with single responsibility principle
2. **Dependency Injection**: Loose coupling through constructor injection
3. **Configuration Management**: Centralized configuration with environment variable overrides
4. **Event-Driven Architecture**: Asynchronous message processing with proper error handling
5. **Repository Pattern**: Database operations abstracted through context manager
6. **Strategy Pattern**: Different agent strategies for different message types

### Data Flow (Refactored with Performance Optimization)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liwi-sys/XianyuAutoAgent](https://github.com/liwi-sys/XianyuAutoAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
