---
trigger: always_on
description: Vetch is an energy-aware observability SDK for LLM inference. It wraps API calls to log energy, cost, and carbon without reading prompt/completion content.
---

# Vetch SDK Development Guidelines

## Project Overview

Vetch is an energy-aware observability SDK for LLM inference. It wraps API calls to log energy, cost, and carbon without reading prompt/completion content.

**License:** Apache 2.0

---

## Core Principles (Non-Negotiable)

### Fail-Open
- If Vetch fails (grid API down, calculation error, patch failure), the LLM call MUST proceed
- Never crash the host application
- Wrap errors, log them, continue

```python
try:
    # Vetch logic
except Exception:
    logger.warning("Vetch failed, continuing without tracking")
    ctx.tracking_disabled = True
# LLM call always proceeds
```

### Fail-Loud
- Every log includes `signal_quality` - no silent degradation
- Stale data? Say so. Missing region? Say so. Patching disabled? Say so.
- Collect warnings during processing, emit in `vetch_warnings` field

```python
# Validation functions return (result, warnings) tuples
def validate_something(data: dict) -> tuple[Result | None, list[str]]:
    warnings: list[str] = []
    if problem:
        warnings.append("Specific warning message")
    return result, warnings

# Wrapper collects warnings throughout lifecycle
self._warnings: list[str] = []
validated, warnings = validate_energy_override(override)
self._warnings.extend(warnings)
# ... emit vetch_warnings=self._warnings in event
```

### Privacy-First
- ZERO access to prompt or completion content
- Only touch: model name, token counts, region, latency
- Never buffer response content

### Observability-Transparent
- Detect prior patches (Datadog, OpenTelemetry, Sentry)
- Forward all attribute access
- Never break the observability chain

---

## Technical Constraints

### Dependencies
- **Runtime**: stdlib only (urllib, contextvars, tempfile, json, etc.)
- **Optional peers**: openai>=1.0,<2.0, google-cloud-aiplatform>=1.0
- **Test**: pytest, hypothesis, pytest-cov

### Python Version
- Minimum: Python 3.9
- Use `Union[X, Y]` not `X | Y` (3.10+ syntax)
- Use `dict[str, int]` not `Dict[str, int]` (3.9 supports lowercase generics)

### Performance
- Under 5ms overhead for synchronous calls
- Zero latency added to TTFT for streaming
- Memory-safe: count streaming chunks, don't accumulate

---

## Key Implementation Patterns

### Wrapper Transparency
```python
# Patches must preserve all attributes
original_func.some_attr  # Must still work after patching
original_func.__name__   # Must be preserved
```

### Stream Handling (Memory-Safe)
```python
# CORRECT: count, don't accumulate
accumulated_chars = 0
for chunk in stream:
    accumulated_chars += len(extract_text(chunk))
    yield chunk  # Pass through immediately
# Emit event in finally block

# WRONG: accumulates in memory
chunks = []
for chunk in stream:
    chunks.append(chunk)  # Memory leak!
    yield chunk
```

### File Locking (Cross-Platform)
```python
# Unix: fcntl.flock()
# Windows: msvcrt.locking()
# Always with timeout (100ms)
```

### Exception Hierarchy
```python
# All Vetch exceptions inherit from VetchError (which inherits ValueError)
class VetchError(ValueError): pass
class RegistryError(VetchError):
    def __init__(self, message: str, model: str | None = None): ...
class ProviderError(VetchError):
    def __init__(self, message: str, provider: str | None = None): ...
class ConfigurationError(VetchError):
    def __init__(self, message: str, field: str | None = None): ...

# Subclasses store contextual fields for debugging
```

### Token Estimation Fallback
When streaming lacks usage data, estimate tokens from character count:
```python
# ~4 characters per token (English text heuristic)
estimated_output_tokens = max(1, accumulated_chars // 4)
# Mark clearly: usage_estimated=True, usage_estimation_method="char_ratio"
```

---

## Kudzu Sandbox (kudzu/)

Kudzu is the local chaos harness for testing Vetch against Ollama. It runs agent scenarios and checks which waste advisories fire.

### Hardware constraints (M5, 16GB unified memory)

- **One run at a time. Never in parallel.** Ollama loads 8B models (~5GB each) into unified memory shared with the GPU. Two simultaneous runs can exhaust available RAM, causing macOS to kill processes and VS Code to crash.
- Stick to one model per session. Switching models forces Ollama to evict and reload (~5GB swap each time).
- Keep `max_steps` under 25 for 8B models.
- Check `ollama ps` before starting a run if you're unsure what's loaded.

### Scientific standard for Vetch changes

Kudzu findings are evidence, not proof. Before changing any Vetch threshold, detection condition, or advisory logic based on a Kudzu observation:

- **Minimum five runs** on the same profile with different seeds, showing a consistent pattern.
- **Explain the failure mode** of the current code mechanistically — which exact condition failed and why.
- **Check the opposing direction**: would the proposed change cause false positives on clean runs?

One experiment that narrowly misses a threshold is a hypothesis, not a justification. Document it in `kudzu/DIARY.md` and run replication seeds before proposing a code change.

### Registry entries (pricing and energy)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prismatic-labs/vetch](https://github.com/prismatic-labs/vetch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
