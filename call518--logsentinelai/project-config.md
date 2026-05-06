---
trigger: always_on
description: **LogSentinelAI** is a declarative, LLM-powered log analysis framework for security, anomaly, and system event detection. The core innovation is **Declarative Extraction**—define a Pydantic model in each analyzer, and the LLM returns structured JSON matching that schema automatically. No manual parsing required.
---

# Copilot Instructions for LogSentinelAI

## Project Overview
**LogSentinelAI** is a declarative, LLM-powered log analysis framework for security, anomaly, and system event detection. The core innovation is **Declarative Extraction**—define a Pydantic model in each analyzer, and the LLM returns structured JSON matching that schema automatically. No manual parsing required.

**Data flow**: Log file input → Parsing → LLM-based analysis → Enrichment (GeoIP, reputation) → Output (JSON, Elasticsearch, stdout) → Visualization (Kibana) → Telegram Alerts.

## Architecture & Key Components
- `src/logsentinelai/analyzers/`: Analyzer modules for each log type. Each follows the pattern: define Pydantic models → get prompts → call `run_generic_*_analysis()`.
- `src/logsentinelai/core/`: Core business logic:
  - `llm.py`: LLM abstraction using **Outlines** library for structured generation
  - `commons.py`: Generic batch/realtime analysis functions, argument parsing, **runtime logging configuration**  
  - `prompts.py`: Centralized prompt templates with detailed security analysis instructions
  - `config.py`: Environment-based configuration loading with **dotenv runtime loading**
  - `elasticsearch.py`: ES integration with **Telegram alert triggers**
  - `monitoring.py`: **Real-time file monitoring** with EOF tracking, rotation detection, and auto-sampling
- `src/logsentinelai/utils/`: Utility functions (general helpers, GeoIP lookup/download, **Telegram alerts**)
- `pyproject.toml`: CLI scripts mapped to analyzer entry points

## Critical Patterns & Conventions

### 1. Declarative Extraction Pattern
**Core concept**: Define result structure as Pydantic models, LLM fills the data automatically.
```python
class SecurityEvent(BaseModel):
    event_type: str
    severity: SeverityLevel
    source_ips: list[str] = Field(description="ALL IPs - NEVER empty")
    related_logs: list[str] = Field(min_length=1, description="Original log lines - at least one required")
    # LLM automatically extracts these fields from raw logs
```

### 2. LLM Integration via Outlines
- Uses **Outlines** library for reliable structured generation
- `core/llm.py` abstracts multiple providers (OpenAI, Ollama, vLLM, Gemini)
- **Gemini special handling**: Raw text → JSON validation due to API limitations with structured output
- All others use `outlines.from_openai()` with structured output
- **Error handling**: Gemini responses cleaned (removes markdown code blocks) before JSON parsing

### 3. Runtime Configuration Loading Pattern
**CRITICAL**: Environment variables must be read at **runtime**, not at module import time.
```python
# ❌ WRONG: Reads at module import (before config loaded)
LOG_FILE = os.getenv("LOG_FILE", "default.log")

# ✅ CORRECT: Reads at function runtime (after config loaded)
def setup_logger():
    log_file = os.getenv("LOG_FILE", "default.log")
```
- `commons.py`: `setup_logger()` reads environment variables at runtime
- `config.py`: Uses `load_dotenv()` to load config files before other modules read env vars
- **Config search order**: `/etc/logsentinelai.config` → `./.env` → error if none found

### 4. Analyzer Structure Pattern
Every analyzer follows this exact pattern:
```python
# 1. Define Pydantic models (SecurityEvent/LogEvent, Statistics, LogAnalysis)
# 2. Import: get_*_prompt, run_generic_*_analysis, create_argument_parser
# 3. main() function with standard argument parsing
# 4. Call run_generic_batch_analysis() or run_generic_realtime_analysis()
```

**Required Pydantic Fields**:
- `related_logs: list[str] = Field(min_length=1)` - Original log entries that triggered event
- `source_ips: list[str]` - **NEVER empty** - extract ALL IPs from logs
- `events: list[Event] = Field(min_length=1)` - **MUST NEVER BE EMPTY** - create at least INFO event if no issues

### 5. Real-time Monitoring Pattern
- **EOF tracking**: `monitoring.py` starts from current file end, processes only new logs
- **File rotation detection**: Uses inode tracking to detect log rotation/truncation  
- **Auto-sampling**: When pending lines > threshold, switches to sampling mode (keeps latest N lines)
- **Chunk timeout**: Forces processing if pending lines don't reach chunk_size within timeout
- **SSH support**: Remote file monitoring via `RemoteSSHLogMonitor`

### 6. Logging Context Pattern
- **Log type injection**: `LOG_TYPE_CTX` context variable adds analyzer type to all log messages
- **Runtime logger setup**: `setup_logger()` reads LOG_LEVEL/LOG_FILE at function call time
- **Consistent format**: `[timestamp] [level] [log_type] (module) message`

### 7. Telegram Integration Pattern
- **Auto-triggers**: CRITICAL severity events OR processing failures
- **Configuration**: `TELEGRAM_ENABLED=true`, `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID`
- **Implementation**: `utils/telegram_alert.py` using `python-telegram-bot`
- **Integration point**: `core/elasticsearch.py` checks events and sends alerts

## Developer Workflows

### Adding New Analyzers
1. Create `src/logsentinelai/analyzers/new_analyzer.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [call518/LogSentinelAI](https://github.com/call518/LogSentinelAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
