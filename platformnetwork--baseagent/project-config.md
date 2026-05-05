---
trigger: always_on
description: This file provides instructions for AI agents working on this codebase. It also serves as a comprehensive guide for building autonomous coding agents.
---

# AGENTS.md - Building a High-Performance Autonomous Agent

This file provides instructions for AI agents working on this codebase. It also serves as a comprehensive guide for building autonomous coding agents.

## Quick Start

```bash
# Install dependencies
pip install -e .

# Run with Term SDK
python agent.py

# Local testing with Term Challenge
git clone https://github.com/PlatformNetwork/term-challenge.git
pip install -e term-challenge/sdk/python/
```

## Project Structure

```
baseagent/
├── agent.py           # Entry point for Term SDK
├── src/
│   ├── core/
│   │   ├── loop.py    # Main agent loop (caching, verification)
│   │   └── compaction.py  # Context management
│   ├── tools/         # Tool implementations
│   ├── prompts/
│   │   └── system.py  # Codex-inspired system prompt
│   └── api/
│       └── client.py  # LLM API client
├── rules/             # Agent development guidelines (READ THESE)
└── astuces/           # Practical techniques used here
```

---

# Part 1: Core Principles

## The Golden Rule: NO HARDCODING

Your agent must solve tasks through **reasoning**, not **pattern matching**.

### FORBIDDEN

```python
# NEVER do this
if "file" in ctx.instruction:
    create_file()
elif "compile" in ctx.instruction:
    compile_code()
```

### REQUIRED

```python
# Always let LLM decide
response = llm.chat([
    {"role": "system", "content": SYSTEM_PROMPT},
    {"role": "user", "content": f"Task: {ctx.instruction}"}
])
execute(response.tool_calls)
```

## What Makes a Generalist Agent

| Characteristic | Description |
|----------------|-------------|
| Single code path | Same logic for ALL tasks |
| LLM-driven decisions | LLM chooses actions, not if-statements |
| No task keywords | Zero references to specific task content |
| Iterative execution | Observe → Think → Act loop |

## The Test

Ask yourself: **"Would this code behave differently if I changed the task instruction?"**

If YES and it's not because of LLM reasoning → it's hardcoding → FORBIDDEN.

---

# Part 2: Architecture

## The Agent Loop

```python
def run_agent_loop(ctx: AgentContext) -> str:
    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": ctx.instruction}
    ]
    
    while True:
        # 1. Apply caching for cost efficiency
        messages = apply_caching(messages)
        
        # 2. Manage context to prevent overflow
        messages = manage_context(messages, max_tokens=180000)
        
        # 3. Call LLM
        response = ctx.llm.chat(messages, tools=TOOLS)
        
        # 4. Check for completion
        if not response.has_tool_calls():
            # Inject verification before completing
            if not verified:
                messages.append(verification_prompt(ctx.instruction))
                verified = True
                continue
            return response.text
        
        # 5. Execute tools
        for call in response.tool_calls:
            result = execute_tool(call)
            messages.append(tool_result(call.id, result))
    
    return "Task completed"
```

## Essential Patterns

### 1. Explore First
Always gather context before acting:
```python
context = shell("pwd && ls -la")
readme = shell("cat README.md 2>/dev/null")
```

### 2. Iterative Execution
Never try to do everything in one shot:
```python
while not done:
    response = llm.chat(messages)
    result = execute(response)
    messages.append(result)
```

### 3. Double Confirmation
Always verify before completing:
```python
if response.says_complete:
    if not already_verified:
        inject_verification_prompt()
        continue
    return complete()
```

---

# Part 3: Key Techniques

## 1. Prompt Caching (90% Cost Reduction)

Cache the **system prompt + last 2 messages** for massive cache hits.

```python
def apply_caching(messages):
    # Cache system messages (stable)
    for msg in messages:
        if msg["role"] == "system":
            add_cache_control(msg)
    
    # Cache last 2 non-system messages (extends prefix)
    non_system = [m for m in messages if m["role"] != "system"]
    for msg in non_system[-2:]:
        add_cache_control(msg)
    
    return messages
```

**Why it works**: Anthropic caches prefixes. Caching the last messages extends the cached prefix to include the entire conversation history.

## 2. Self-Verification

Before completing, force the agent to verify its work:

```python
VERIFICATION_PROMPT = f"""
STOP - Before completing, verify your work:

Original instruction: {ctx.instruction}

Checklist:
1. Re-read the instruction above
2. List ALL requirements (explicit and implicit)
3. Run commands to verify each requirement
4. Only complete after ALL verifications pass

You are in headless mode - do NOT ask questions.
"""
```

## 3. Context Management

Prevent token overflow with pruning and compaction:

```python
def manage_context(messages, max_tokens):
    current = estimate_tokens(messages)
    
    # Stage 1: Prune old tool outputs
    if current > max_tokens * 0.70:
        messages = prune_tool_outputs(messages, keep_last=5)
    
    # Stage 2: AI compaction
    if current > max_tokens * 0.85:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlatformNetwork/baseagent](https://github.com/PlatformNetwork/baseagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
