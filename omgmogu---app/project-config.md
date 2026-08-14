---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Xianyu (闲鱼) automated commenting assistant - a locally-run automation tool that uses Appium and AI to intelligently generate and post comments on Xianyu (Chinese second-hand marketplace) product pages. The system is designed for educational and research purposes only.

**IMPORTANT**: This tool is for learning/research only and must comply with platform terms of service. Users bear all responsibility and risks.

## Development Environment Setup

### Prerequisites
- Python 3.8+
- Java JDK 8+
- Android SDK with platform-tools and tools
- Node.js 16+ with Appium Server 2.0+
- Android device/emulator (API 23+) with Xianyu app

### Environment Setup Commands
```bash
# Install dependencies
pip install -r requirements.txt

# Set up virtual environment (recommended)
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

# Verify Android connection
adb devices

# Start Appium server (in separate terminal)
appium

# Run the application
cd src
python main.py
```

### Common Development Commands
```bash
# Check system dependencies and configuration
cd src && python -c "from config import config; print('Config loaded successfully')"

# Test database connectivity
cd src && python -c "from database import get_db; db = get_db(); print('Database initialized')"

# Test DeepSeek API connection (requires API key in config)
cd src && python -c "from ai_client import test_deepseek_connection; import asyncio; asyncio.run(test_deepseek_connection())"

# View application logs
tail -f data/logs/xianyu_assistant.log

# Check Appium server status
curl http://localhost:4723/wd/hub/status

# Debug Android device connection
adb logcat | grep -i xianyu

# Run with specific device (if multiple devices connected)
adb devices  # Get device ID first
# Then edit config/settings.yaml to set udid
```

### Configuration
- Edit `config/settings.yaml` to set DeepSeek API key and device settings
- The system requires manual login to Xianyu app before automation

## Key File Structure

### Core Source Files (`src/`)
- `main.py` - Main orchestrator class `XianyuCommentAssistant` with threading and task management
- `app_controller.py` - Appium WebDriver wrapper `XianyuAppController` for mobile app interaction
- `product_analyzer.py` - Product info extraction `XianyuProductAnalyzer` with multiple selector strategies
- `comment_generator.py` - Hybrid AI+template comment generation with quality filtering
- `comment_publisher.py` - Anti-detection comment publishing with human behavior simulation
- `ai_client.py` - DeepSeek API client with async/sync support and retry logic
- `database.py` - SQLite operations with thread-safe connection pooling
- `config.py` - YAML configuration management with validation

### Configuration Files
- `config/settings.yaml` - Main configuration including Appium capabilities, DeepSeek API, anti-detection settings, UI selectors, and comment templates

### Data Storage
- `data/logs/` - Application logs with rotation
- `data/*.db` - SQLite databases for products, comments, tasks, and statistics

## Architecture Overview

The system follows a modular pipeline architecture:

```
Manual Login → APP Control → Product Analysis → Comment Generation → Comment Publishing → Data Storage
```

### Core Components

**XianyuCommentAssistant (main.py)**: Main orchestrator that coordinates all modules and manages task execution in separate threads.

**XianyuAppController (app_controller.py)**: Handles Appium WebDriver connections, page navigation, and element interactions with the Xianyu Android app.

**XianyuProductAnalyzer (product_analyzer.py)**: Extracts product information (title, price, seller, condition, existing comments) from product pages using various selector strategies.

**XianyuCommentGenerator (comment_generator.py)**: Generates comments using hybrid AI+template approach. Supports multiple comment types (inquiry, interest, compliment, comparison, concern) with quality filtering.

**XianyuCommentPublisher (comment_publisher.py)**: Publishes comments with anti-detection mechanisms (human-like typing, random intervals, frequency limits).

**DeepSeekClient (ai_client.py)**: Async HTTP client for DeepSeek AI API with retry logic, rate limiting, and error handling.

**Database (database.py)**: SQLite-based storage for products, comments, tasks, statistics, and error logs with thread-safe operations.

**Config (config.py)**: YAML-based configuration management with validation and default value handling.

### Data Flow
1. **Initialization**: Load config, connect to app, initialize AI client
2. **Task Processing**: For each product URL:
   - Navigate to product page
   - Extract product information and existing comments
   - Generate personalized comments based on product characteristics
   - Publish comments with anti-detection delays
   - Store results and statistics
3. **Monitoring**: Real-time progress tracking, error handling, and cleanup

### Anti-Detection Features
- Randomized typing speeds (50-150ms per character)
- Variable comment intervals (15-45 seconds)
- Daily/hourly limits (configurable)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omgmogu/-app](https://github.com/omgmogu/-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
