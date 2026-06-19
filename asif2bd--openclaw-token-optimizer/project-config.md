---
trigger: always_on
description: Reduce OpenClaw token usage and API costs through smart model routing, heartbeat optimization, budget tracking, and native 2026.2.15 features (session pruning, bootstrap size limits, cache TTL alignment). Use when token costs are high, API rate limits are being hit, or hosting multiple agents at scale. The 4 executable scripts (context_optimizer, model_router, heartbeat_optimizer, token_tracker) are local-only — no network requests, no subprocess calls, no system modifications. Reference files (
---


# Token Optimizer

Comprehensive toolkit for reducing token usage and API costs in OpenClaw deployments. Combines smart model routing, optimized heartbeat intervals, usage tracking, and multi-provider strategies.

## Quick Start

**Immediate actions** (no config changes needed):

1. **Generate optimized AGENTS.md (BIGGEST WIN!):**
   ```bash
   python3 scripts/context_optimizer.py generate-agents
   # Creates AGENTS.md.optimized — review and replace your current AGENTS.md
   ```

2. **Check what context you ACTUALLY need:**
   ```bash
   python3 scripts/context_optimizer.py recommend "hi, how are you?"
   # Shows: Only 2 files needed (not 50+!)
   ```

3. **Install optimized heartbeat:**
   ```bash
   cp assets/HEARTBEAT.template.md ~/.openclaw/workspace/HEARTBEAT.md
   ```

4. **Enforce cheaper models for casual chat:**
   ```bash
   python3 scripts/model_router.py "thanks!"
   # Single-provider Anthropic setup: Use Sonnet, not Opus
   # Multi-provider setup (OpenRouter/Together): Use Haiku for max savings
   ```

5. **Check current token budget:**
   ```bash
   python3 scripts/token_tracker.py check
   ```

**Expected savings:** 50-80% reduction in token costs for typical workloads (context optimization is the biggest factor!).

## Core Capabilities

### 1. Context Optimization (NEW!)

**Biggest token saver** — Only load files you actually need, not everything upfront.

**Problem:** Default OpenClaw loads ALL context files every session:
- SOUL.md, AGENTS.md, USER.md, TOOLS.md, MEMORY.md
- docs/**/*.md (hundreds of files)
- memory/2026-*.md (daily logs)
- Total: Often 50K+ tokens before user even speaks!

**Solution:** Lazy loading based on prompt complexity.

**Usage:**
```bash
python3 scripts/context_optimizer.py recommend "<user prompt>"
```

**Examples:**
```bash
# Simple greeting → minimal context (2 files only!)
context_optimizer.py recommend "hi"
→ Load: SOUL.md, IDENTITY.md
→ Skip: Everything else
→ Savings: ~80% of context

# Standard work → selective loading
context_optimizer.py recommend "write a function"
→ Load: SOUL.md, IDENTITY.md, memory/TODAY.md
→ Skip: docs, old memory, knowledge base
→ Savings: ~50% of context

# Complex task → full context
context_optimizer.py recommend "analyze our entire architecture"
→ Load: SOUL.md, IDENTITY.md, MEMORY.md, memory/TODAY+YESTERDAY.md
→ Conditionally load: Relevant docs only
→ Savings: ~30% of context
```

**Output format:**
```json
{
  "complexity": "simple",
  "context_level": "minimal",
  "recommended_files": ["SOUL.md", "IDENTITY.md"],
  "file_count": 2,
  "savings_percent": 80,
  "skip_patterns": ["docs/**/*.md", "memory/20*.md"]
}
```

**Integration pattern:**
Before loading context for a new session:
```python
from context_optimizer import recommend_context_bundle

user_prompt = "thanks for your help"
recommendation = recommend_context_bundle(user_prompt)

if recommendation["context_level"] == "minimal":
    # Load only SOUL.md + IDENTITY.md
    # Skip everything else
    # Save ~80% tokens!
```

**Generate optimized AGENTS.md:**
```bash
context_optimizer.py generate-agents
# Creates AGENTS.md.optimized with lazy loading instructions
# Review and replace your current AGENTS.md
```

**Expected savings:** 50-80% reduction in context tokens.

### 2. Smart Model Routing (ENHANCED!)

Automatically classify tasks and route to appropriate model tiers.

**NEW: Communication pattern enforcement** — Never waste Opus tokens on "hi" or "thanks"!

**Usage:**
```bash
python3 scripts/model_router.py "<user prompt>" [current_model] [force_tier]
```

**Examples:**
```bash
# Communication (NEW!) → ALWAYS Haiku
python3 scripts/model_router.py "thanks!"
python3 scripts/model_router.py "hi"
python3 scripts/model_router.py "ok got it"
→ Enforced: Haiku (NEVER Sonnet/Opus for casual chat)

# Simple task → suggests Haiku
python3 scripts/model_router.py "read the log file"

# Medium task → suggests Sonnet
python3 scripts/model_router.py "write a function to parse JSON"

# Complex task → suggests Opus
python3 scripts/model_router.py "design a microservices architecture"
```

**Patterns enforced to Haiku (NEVER Sonnet/Opus):**

*Communication:*
- Greetings: hi, hey, hello, yo
- Thanks: thanks, thank you, thx
- Acknowledgments: ok, sure, got it, understood
- Short responses: yes, no, yep, nope
- Single words or very short phrases

*Background tasks:*
- Heartbeat checks: "check email", "monitor servers"
- Cronjobs: "scheduled task", "periodic check", "reminder"
- Document parsing: "parse CSV", "extract data from log", "read JSON"
- Log scanning: "scan error logs", "process logs"

**Integration pattern:**
```python
from model_router import route_task

user_prompt = "show me the config"
routing = route_task(user_prompt)

if routing["should_switch"]:
    # Use routing["recommended_model"]
    # Save routing["cost_savings_percent"]
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Asif2BD/OpenClaw-Token-Optimizer](https://github.com/Asif2BD/OpenClaw-Token-Optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
