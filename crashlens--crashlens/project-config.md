---
trigger: always_on
description: **CrashLens** is a privacy-first AI token waste detection CLI tool that analyzes Langfuse JSONL logs to identify and prevent costly patterns in LLM API usage. It runs 100% locally with no data egress.
---

# CrashLens - AI Coding Agent Instructions

## 🎯 Project Overview

**CrashLens** is a privacy-first AI token waste detection CLI tool that analyzes Langfuse JSONL logs to identify and prevent costly patterns in LLM API usage. It runs 100% locally with no data egress.

**Core Mission**: Detect retry loops, fallback storms, model overkill, and fallback failures before they burn through your AI budget.

**Tech Stack**: Python 3.12+, Poetry, Click CLI framework, pytest

**Entry Point**: `crashlens/cli.py` (2687 lines) → `@click.group()` decorator pattern

---

## 🏗️ Architecture Overview

### Pipeline Pattern (Chain of Responsibility)
```
User Command (Click CLI)
    ↓
Input Source (file/stdin/clipboard/Langfuse API/Helicone API)
    ↓
LangfuseParser (JSONL → normalized traces grouped by traceId)
    ↓
Detector Pipeline (4 parallel detectors with priority-based suppression)
    ├─ RetryLoopDetector (exact string matching, exponential backoff detection)
    ├─ FallbackStormDetector (cascade detection within time windows)
    ├─ OverkillModelDetector (model vs. task suitability scoring)
    └─ FallbackFailureDetector (failed fallback chain detection)
    ↓
Policy Engine (optional YAML rule evaluation)
    ├─ PolicyRule.evaluate() per log entry (hot loop)
    └─ Constant-memory stats collection (conditional, <10% overhead)
    ↓
Formatters (multiple output formats)
    ├─ MarkdownFormatter (readable reports)
    ├─ JSONFormatter (detailed output by category)
    └─ SlackFormatter (Slack block kit messages)
    ↓
Output (file/stdout/Slack webhook)
```

### Module Organization
```
crashlens/
├── cli.py                          # Main entry point, all Click commands
├── parsers/langfuse.py            # JSONL parsing with schema validation
├── detectors/                     # Waste pattern detection
│   ├── retry_loops.py             # Exact string matching, exponential backoff
│   ├── fallback_storm.py          # Cascade detection in time windows
│   ├── overkill_model_detector.py # Model suitability scoring
│   └── fallback_failure.py        # Failed fallback chain detection
├── policy/                        # Rule evaluation system
│   ├── engine.py                  # PolicyEngine with hot loop instrumentation
│   └── templates/                 # Reusable YAML policy templates
├── formatters/                    # Output rendering
│   ├── markdown_formatter.py      # Human-readable Markdown
│   ├── json_formatter.py          # Structured JSON with category breakdown
│   └── slack_formatter.py         # Slack Block Kit messages
├── pii/                           # Privacy-first PII removal
│   ├── sanitizer.py               # Email, phone, SSN scrubbing
│   └── patterns.py                # PII regex patterns
├── config/                        # YAML config schemas
└── utils/                         # Shared utilities
```

---

## 🔑 Key Conventions

### 1. Click CLI Pattern
**Always use decorators for commands:**
```python
@click.command()
@click.option('--format', type=click.Choice(['slack', 'markdown', 'json']), default='slack')
@click.option('--config', type=click.Path(exists=True, path_type=Path), help="Custom config")
@click.argument('logfile', type=click.Path(exists=True, path_type=Path), required=False)
def scan(logfile: Optional[Path], format: str, config: Optional[Path]) -> None:
    """Scan logs for token waste patterns."""
    # Implementation
```

**Error handling convention:**
```python
if error_condition:
    click.echo(click.style(f"❌ Error: {message}", fg="red"), err=True)
    sys.exit(1)
```

### 2. Detector Interface
**All detectors must implement:**
```python
class MyDetector:
    def __init__(self, threshold: int, time_window_minutes: int):
        """Configure detection parameters."""
        pass
    
    def detect(
        self,
        traces: Dict[str, List[Dict[str, Any]]],  # traceId -> list of records
        model_pricing: Optional[Dict[str, Any]] = None,
        already_flagged_ids: Optional[set] = None,  # For suppression
    ) -> List[Dict[str, Any]]:
        """
        Returns list of detection dicts with structure:
        {
            'trace_id': str,
            'detector': str,  # Human-readable name
            'waste_cost': float,
            'waste_tokens': int,
            'severity': 'high' | 'medium' | 'low',
            'description': str,
            'suggestion': str,
            'records': List[Dict[str, Any]],  # Relevant log entries
            # Detector-specific fields...
        }
        """
        detections = []
        # Detection logic
        return detections
```

**Suppression Pattern** (avoid double-counting):
```python
if trace_id in already_flagged_ids:
    continue  # Higher-priority detector already claimed this trace
```

### 3. Policy YAML Format
**Standard structure:**
```yaml
version: 1
global:
  max_violations_per_rule: 100  # Circuit breaker

rules:
  - id: excessive_retries
    description: "Block traces with >3 retries"
    match:
      retry_count: ">3"                      # Operators: >, >=, <, <=, ==, in:[...], regex:...
      metadata.fallback_attempted: true

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crashlens/crashlens](https://github.com/Crashlens/crashlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
