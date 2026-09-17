---
trigger: always_on
description: handles_kinds={KIND_TEXT}, priority=50,
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **production-grade DingTalk digital employee harness** that connects DingTalk messaging to OpenCode's AI brain. It's a template for building AI agents that can:
- Listen to group/private/@ messages in DingTalk
- Process text, images, and files with multimodal AI
- Reply intelligently using free OpenCode models
- Run 24/7 with self-healing daemon processes

The project follows a **core/custom separation pattern** where:
- `src/core/`, `bin/core/`, `tests/core/` contain the harness framework (don't modify)
- `src/custom/`, `bin/custom/`, `tests/custom/` contain DingTalk-specific implementations (modify here)
- `config/*.local.*` contain sensitive credentials (gitignored)

**IMPORTANT**: Read [AGENTS.md](./AGENTS.md) for comprehensive project instructions, layer boundaries, and detailed implementation guidance. AGENTS.md is the authoritative reference for AI agents working on this codebase.

## Core Architecture

Full architecture details in [ARCHITECTURE.md](./ARCHITECTURE.md) (整体架构图 + 13 个最佳实践).

### Three-Layer Boundary (详见 AGENTS.md)

- **@core** (`src/core/`, `bin/core/`, `tests/core/`) — Harness framework, DON'T modify (bug fixes → PR to upstream)
- **@custom** (`src/custom/`, `bin/custom/`, `tests/custom/`) — DingTalk-specific, modify freely here
- **@config** (`config/*.local.*`) — Sensitive credentials, gitignored

### Data Flow

See "数字员工架构图" in [README.md](./README.md) for complete flow diagram.

```
DingTalk → dws consume → event_watcher.py → Capability plugins → opencode serve → dws send → DingTalk
            (bridge)      (log-tail)         (registered)        (brain)          (replier)
```

## Common Commands

### Service Management

```bash
# Start all services (auto-detects launchd/nohup mode)
bash bin/core/start.sh

# Stop all services
bash bin/core/stop.sh

# Restart (also triggered by /reboot in chat)
bash bin/core/reboot.sh

# Health check (6 checks: connect/log/serve/http)
bash bin/core/healthcheck.sh
```

### Testing

```bash
# Shell unit tests (syntax + function-level assertions)
bash tests/core/unit_test.sh

# Python unit tests (all)
for t in tests/core/*.py tests/custom/*.py; do python3 "$t"; done

# Specific test
python3 tests/core/test_agent_common.py
python3 tests/custom/test_ack_capability.py

# E2E tests (require real DingTalk setup)
bash tests/custom/e2e_text_http_test.sh      # HTTP brain test (no DingTalk needed)
bash tests/custom/e2e_at_test.sh             # @ mention subscription test
```

### Configuration

```bash
# Copy templates and fill real values
cp config/constants.sh config/constants.local.sh
# Edit *.local.sh with:
#   DWS_EVENT_GROUP, DWS_PROFILE, AGENT_PROFILE (must match DWS_PROFILE)
#   AGENT_BRAIN=opencode, AGENT_OPENCODE_MODEL=opencode/deepseek-v4-flash-free
#   PATH must include ~/.local/bin (dws) and ~/.opencode/bin (opencode)

# Verify dws authentication
dws auth status
dws profile list
```

### Development

```bash
# Watch logs in real-time
tail -f monitor.log agent-connect.log opencode.log

# Check service status
pgrep -fl "monitor.sh|dws-connect|event_watcher.py|opencode serve"

# Manual component testing
source config/constants.local.sh
python3 src/core/event_watcher.py            # Run event watcher in foreground
bash bin/custom/dws-connect.sh               # Run DingTalk connector
```

## Key Design Patterns

### 1. Capability Plugin System

All features are capabilities registered via `Capability(...)`. Example:

```python
from core.capabilities import Capability, register
from core.inbound import KIND_TEXT

def on_inbound(msg):
    return True  # True = consumed, False = pass to next

register(Capability(name="my_cap", on_inbound=on_inbound,
                    handles_kinds={KIND_TEXT}, priority=50,
                    dedup=True, loop_guard=True))
```

Registered in `src/custom/capabilities/__init__.py`, controlled by `CAP_<NAME>_ENABLED`.

### 2. Session Management

- **Session reuse** (`AGENT_SESSION_REUSE=1`): Multi-turn context per conversation
- **TTL expiry** (`AGENT_SESSION_TTL=1800`): Idle 30min → rebuild
- **LRU eviction** (`AGENT_SESSION_MAX=64`): Max concurrent sessions
- **Reset keywords** (`AGENT_SESSION_RESET_KEYWORDS="/new,新话题"`): User triggers context clear
- **Per-turn model switch bypasses reuse** (`AGENT_OPENCODE_MODEL_FLASH`, #117): a turn whose model differs from `AGENT_OPENCODE_MODEL` runs in its own one-shot session (created → posted → deleted) instead of joining the reused one. Provider prompt cache is per-model, so switching models inside a long reused session re-encodes the whole history (measured: `input` 303 → 57,557 in one turn). Trade-off: that turn sees no prior context. See ARCHITECTURE.md §8.
- Credentials via `find_serve_credentials()` → caches to `.serve.{pid,port,pwd}`

### 3. Daemon + Self-Healing

See ARCHITECTURE.md "launchd 守护" for full details. Key points:

- `monitor.sh`: cleanup → start components → healthcheck loop → circuit breaker
- `healthcheck.sh`: 6 checks (4 hard failures + 2 warnings)
- After `MAX_FAILURES` consecutive failures → exit 0 (notify + wait for manual fix)

### 4. Testing Strategy

See AGENTS.md "测试约定" for complete testing patterns:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hugozhu/dingtalk-opencode-tag](https://github.com/hugozhu/dingtalk-opencode-tag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
