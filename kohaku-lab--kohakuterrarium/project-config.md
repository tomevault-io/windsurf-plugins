---
trigger: always_on
description: A universal agent framework for building any type of fully self-driven agent system.
---

# KohakuTerrarium

A universal agent framework for building any type of fully self-driven agent system.

## Project Overview

KohakuTerrarium is a Python framework that enables building any kind of agent system - from SWE agents like Claude Code to conversational bots like Neuro-sama to autonomous monitoring systems. The name "Terrarium" reflects how the framework allows you to build different self-contained agent ecosystems.

## Code Conventions

### File Organization
- Source code: `src/kohakuterrarium/`
- Frontend (Vue 3): `src/kohakuterrarium-frontend/`
- Creature templates: `creatures/`
- Terrarium templates: `terrariums/`
- Examples: `examples/` (agent-apps, terrariums, plugins, code)
- Documentation: `docs/` (en, zh-CN, zh-TW)
- Max lines per file: 600 (hard max: 1000, enforced by `tests/unit/test_file_sizes.py`)
- Highly modularized - one responsibility per module

### Import Rules
1. No imports inside functions (except optional dep and lazy import to avoid long init time)
2. Import grouping order:
   - Built-in modules
   - Third-party packages
   - KohakuTerrarium modules
3. Import ordering within groups:
   - `import` statements before `from` imports
   - Shorter paths before longer paths (by dot count)
   - Alphabetical order (a-z)

### Python Style
- Target: Python 3.10+ (CI matrix runs 3.10 through 3.14)
- Use modern type hints: `list`, `tuple`, `dict`, `X | None` (NOT `List`, `Tuple`, `Dict`, `Optional`, `Union`)
- Prefer `match-case` over deeply nested `if-elif-else`
- Full asyncio throughout (mark sync modules as "require blocking" or "can be to_thread")
- Practical dependencies allowed (pydantic, httpx, rich, etc.)

### Frontend Style
- Vue 3 + Vite, JavaScript only (no TypeScript)
- Run `npm run format:check` and `npm run build` before committing

### Development Setup
- Use `uv pip install -e ".[dev]"` for editable install
- **Never use `sys.path.insert` hacks** in examples or tests - always rely on proper package install
- Examples and tests should import from `kohakuterrarium.*` directly

### Logging (No print!)
- **Avoid naive `print()` in library code** - use structured logging
- Use custom logger based on `logging` module (NOT loguru)
- Format: `[HH:MM:SS] [module.name] [LEVEL] message`
- Color coding: DEBUG=gray, INFO=green, WARNING=yellow, ERROR=red
- **Avoid reserved LogRecord attributes** in extra kwargs: `name`, `msg`, `args`, `levelname`, `levelno`, `pathname`, `filename`, `module`, `lineno`, `funcName`, `created`, `msecs`, `relativeCreated`, `thread`, `threadName`, `process`, `processName`, `message`
- Exception: Test suites (`tests/`) can use simpler output

### Post-impl tasks
1. Verify all your impl follow the rules (ESPECIALLY in-function import!)
2. `black src/ tests/` and `ruff check src/ tests/`
3. Ensure new stuff has corresponding test-suite
4. Logically separated git commits and push (user may explicitly say "draft" — if so, don't push)

## Core Architecture Concepts (CRITICAL)

### Creature vs Terrarium vs Root Agent

**Creature**: A self-contained agent. Has its own LLM, tools, sub-agents, memory, I/O.
Works standalone. Does NOT know it is in a terrarium. Sub-agents inside a creature
are VERTICAL hierarchy (internal delegation, invisible to outside).

**Terrarium**: Pure wiring layer. NO LLM, NO intelligence, NO decision-making.
Loads standalone creature configs (unchanged), creates channels between them,
injects ChannelTriggers, manages lifecycle. That's ALL it does.

**Root Agent**: A creature that sits OUTSIDE the terrarium. Has terrarium management
tools (create, stop, send, observe, hot-plug). The user talks to root; root orchestrates
the terrarium from above. Root is NEVER a peer of terrarium creatures.

```
User <-> Root Agent (creature with terrarium tools)
              |
              v  (creates, manages, observes via tools)
         +-----------+
         | Terrarium |  <-- pure wiring, no intelligence
         +-----------+
         | swe | reviewer | ... |  <-- opaque creatures
```

**Two composition levels (never mix them):**
- VERTICAL (inside creature): controller -> sub-agents (private, hierarchical)
- HORIZONTAL (terrarium): creature <-> creature via channels (peer, opaque)

### Terrarium Config: Optional Root

```yaml
terrarium:
  root:                    # Optional: root agent sits OUTSIDE
    config: creatures/root
    interface: tui
  creatures: [...]         # These run INSIDE the terrarium
  channels: [...]
```

When root is present, it is force-given all terrarium tools and bound to this
terrarium's runtime. It is the user-facing interface.

## Architecture Overview

### Key Design Principle: Controller as Orchestrator

**The controller's role is to dispatch tasks, not to do heavy work itself.**

- Controller outputs should be SHORT: tool calls, sub-agent dispatches, status updates
- Long outputs (user-facing content) should come from **output sub-agents**
- This keeps controller lightweight, fast, and focused on decision-making

### Five Major Systems
1. **Input** - Explicit input that triggers the agent (user request, ASR, group chat message)
2. **Trigger** - Automatic system that triggers agent (timers, events, conditions, composites)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kohaku-Lab/KohakuTerrarium](https://github.com/Kohaku-Lab/KohakuTerrarium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
