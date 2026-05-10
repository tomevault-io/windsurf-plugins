---
trigger: always_on
description: > Pure async Python library for multi-channel conversations with rooms, hooks, and pluggable backends.
---

# RoomKit

> Pure async Python library for multi-channel conversations with rooms, hooks, and pluggable backends.

## Related Repositories

The RoomKit ecosystem spans three repos (sibling directories at `../`):

| Repo | Path | Purpose |
|------|------|---------|
| **roomkit** | `.` (this repo) | Library source, tests, examples |
| **roomkit-docs** | `../roomkit-docs/` | MkDocs documentation site (42 pages: features, architecture, guides, API reference) |
| **roomkit-specs** | `../roomkit-specs/` | Normative RFC specification (`roomkit-rfc.md`) |

### RFC Conformance

The RFC at `../roomkit-specs/roomkit-rfc.md` is the **normative specification**. All implementations must conform to it. Key invariants enforced by the RFC:

- **Inbound pipeline order** (Section 10.1): route → handle_inbound → identity → lock → idempotency → index → BEFORE_BROADCAST → store → broadcast → AFTER_BROADCAST → unlock. MUST NOT reorder.
- **Permission model** (Section 7): access (READ_WRITE/READ_ONLY/WRITE_ONLY/NONE), muting (suppresses responses, not side effects), visibility filtering.
- **Event indexing**: sequential, atomic, monotonically increasing per room, starting at 0.
- **Chain depth**: default max=5. MUST be enforced to prevent AI-to-AI infinite loops.
- **Voice pipeline** (Section 12): stage ordering, AEC double-feeding prevention, DTMF parallel execution before AEC/AGC/denoiser, recording consent, turn detection latency <200ms.
- **Side effects always collected**: tasks and observations are captured regardless of mute/access. "Muting silences the voice, not the brain."
- **4 conformance levels**: Level 0 (Core, REQUIRED), Level 1 (Transport, RECOMMENDED), Level 2 (Rich, OPTIONAL), Level 3 (Voice, OPTIONAL).

If a proposed change conflicts with the RFC, update the RFC first (in `roomkit-specs`), then update the code.

### Documentation

The docs at `../roomkit-docs/` are a MkDocs Material site. When adding features, update the corresponding docs page. Key structure:
- `docs/features.md` — comprehensive feature reference
- `docs/architecture.md` — architecture overview
- `docs/guides/` — practical guides (resampler, sherpa-onnx, smart-turn, WAV recorder, RTP, SIP)
- `docs/api/` — 27 API reference pages (generated via mkdocstrings)

## Quick Reference

```bash
# Install dependencies
uv sync --extra dev

# Run all checks (lint + typecheck + security + test)
make all

# Run specific checks
uv run ruff check src/roomkit/         # Lint check
uv run ruff check src/roomkit/ --fix   # Lint fix
uv run ruff format src/ tests/         # Format code
uv run mypy src/roomkit/               # Type check (enforced in CI)
uv run bandit -r src/ -c pyproject.toml # Security scan (enforced in CI)

# Run tests
uv run pytest tests/ -q                # All tests
uv run pytest tests/test_framework.py -v  # Specific test file
uv run pytest --cov=roomkit --cov-report=term-missing  # With coverage
```

## Project Structure

```
src/roomkit/
├── __init__.py              # Public API exports — ALL public classes exported here
├── _version.py              # Version string (auto-managed)
├── ai_docs.py               # AI documentation helpers (llms.txt, AGENTS.md)
├── core/
│   ├── framework.py         # RoomKit class — central orchestrator
│   ├── _inbound.py          # Inbound message processing pipeline
│   ├── _room_lifecycle.py   # Room CRUD, timers, participant resolution
│   ├── _channel_ops.py      # Channel attach/detach/mute/access/visibility
│   ├── _helpers.py          # Shared helper methods
│   ├── hooks.py             # HookEngine, HookRegistration
│   ├── event_router.py      # Broadcast routing to channels
│   ├── inbound_router.py    # InboundRoomRouter ABC, DefaultInboundRoomRouter
│   ├── locks.py             # RoomLockManager ABC, InMemoryLockManager
│   ├── circuit_breaker.py   # CircuitBreaker (closed/open/half-open)
│   ├── rate_limiter.py      # TokenBucketRateLimiter
│   ├── retry.py             # retry_with_backoff() with RetryPolicy
│   ├── transcoder.py        # DefaultContentTranscoder
│   └── router.py            # ContentTranscoder ABC
├── channels/
│   ├── __init__.py          # Factory functions: SMSChannel(), EmailChannel(), etc.
│   ├── base.py              # Channel ABC
│   ├── transport.py         # TransportChannel (generic transport wrapper)
│   ├── ai.py                # AIChannel (intelligence layer)
│   ├── voice.py             # VoiceChannel (real-time audio with STT/TTS)
│   ├── _voice_hooks.py      # Voice-specific hook implementations
│   ├── _voice_stt.py        # Speech-to-text orchestration
│   ├── _voice_tts.py        # Text-to-speech orchestration
│   ├── _voice_turn.py       # Turn-taking logic for voice channels
│   ├── realtime_voice.py    # RealtimeVoiceChannel (speech-to-speech AI)
│   └── websocket.py         # WebSocketChannel (bidirectional real-time)
├── providers/
│   ├── ai/                  # AIProvider ABC, AIContext, AIResponse, MockAIProvider
│   ├── anthropic/           # AnthropicAIProvider, AnthropicConfig
│   ├── openai/              # OpenAIAIProvider, OpenAIConfig
│   ├── gemini/              # GeminiAIProvider, GeminiConfig
│   ├── mistral/             # MistralAIProvider, MistralConfig

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roomkit-live/roomkit](https://github.com/roomkit-live/roomkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
