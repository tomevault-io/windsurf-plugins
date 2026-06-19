---
trigger: always_on
description: CLI interface to ChatGPT. Send prompts and get responses via command line, with support for GPT-5.2 thinking modes (Auto, Instant, Thinking, Pro) and legacy models (o3, GPT-4.5). Upload files and images for analysis/vision. Continue existing conversations, start new chats, search/list/export/delete/archive history, extract code blocks, toggle temp chat and web search. Uses stealth browser with Chrome auth for authenticated access.
---


# ChatGPT CLI Skill

Query ChatGPT from the command line using your existing Chrome authentication. Supports GPT-5.2 with 4 thinking modes (Auto, Instant, Thinking, Pro) and legacy models (o3, GPT-4.5) behind a submenu. Upload files for analysis and images for vision. Continue existing conversations, start fresh chats, search/export/delete/archive history, extract code blocks, use temporary chats, and toggle web search.

## When to Use This Skill

Trigger when user:
- Wants to query ChatGPT/OpenAI
- Mentions "ask ChatGPT", "prompt GPT", "ChatGPT says"
- Needs extended reasoning on complex problems
- Wants to compare responses between Claude and ChatGPT
- Specifically requests GPT-5.2 Pro, o3, or other OpenAI models
- Wants to list or browse ChatGPT conversations/chat history
- Wants to retrieve or read a specific ChatGPT conversation
- Asks "what did ChatGPT say about..." or wants to find a previous chat
- Wants to continue an existing ChatGPT conversation with a follow-up
- Wants to start a fresh/new ChatGPT conversation
- Wants to export or save a ChatGPT conversation
- Wants to delete or archive a ChatGPT conversation
- Wants to extract just the code from a ChatGPT response
- Wants to use temporary/ephemeral chat mode
- Wants to enable or disable web search in ChatGPT
- Wants to search ChatGPT conversations
- Wants to see ChatGPT projects
- Wants to upload a file to ChatGPT for analysis
- Wants to send an image to ChatGPT for vision/description
- Says "have ChatGPT look at this file" or "analyze this with GPT"

## Prerequisites

1. **Logged into ChatGPT in Chrome**: The skill uses your Chrome session
2. **ChatGPT Pro subscription**: Required for GPT-5.2 Pro access (optional for other models)

## Critical: Always Use run.py Wrapper

**NEVER call scripts directly. ALWAYS use `python3 scripts/run.py [script]`:**

```bash
# CORRECT:
python3 scripts/run.py chatgpt.py --prompt "Your question here"

# WRONG:
python3 scripts/chatgpt.py --prompt "..."  # Fails without venv!
```

## Core Usage

### Basic Query (uses GPT-5.2 Auto by default)
```bash
cd ~/.claude/skills/chatgpt-cli
python3 scripts/run.py chatgpt.py --prompt "Explain quantum entanglement" --show-browser
```

### With Different Models
```bash
# Auto mode — ChatGPT decides how long to think (default)
python3 scripts/run.py chatgpt.py --prompt "Hello" --model auto

# Instant — answers right away, no thinking
python3 scripts/run.py chatgpt.py --prompt "What is 2+2?" --model instant

# Thinking — thinks longer for better answers
python3 scripts/run.py chatgpt.py --prompt "Analyze this algorithm" --model thinking

# Pro — research-grade intelligence, up to 30 min
python3 scripts/run.py chatgpt.py --prompt "Prove this theorem" --model pro

# Legacy model (o3 reasoning)
python3 scripts/run.py chatgpt.py --prompt "Solve this step by step" --model o3
```

### Continue an Existing Conversation
```bash
# By index from --list-chats (0 = most recent)
python3 scripts/run.py chatgpt.py --continue-chat idx-0 --prompt "Follow up on that" --show-browser

# By conversation UUID
python3 scripts/run.py chatgpt.py --continue-chat "abc123-def456-..." --prompt "Tell me more"

# By title substring (case-insensitive)
python3 scripts/run.py chatgpt.py --continue-chat "quantum" --prompt "What about entanglement?"
```

### Start a New Chat
```bash
# Force a fresh conversation (ignores any existing chat context)
python3 scripts/run.py chatgpt.py --new-chat --prompt "Start fresh: explain relativity" --show-browser
```

### With Options
```bash
# Custom timeout for very complex reasoning
python3 scripts/run.py chatgpt.py --prompt "Complex math proof" --timeout 2400 --show-browser

# Save screenshot
python3 scripts/run.py chatgpt.py --prompt "Design a system" --screenshot /tmp/chatgpt.png

# JSON output for parsing
python3 scripts/run.py chatgpt.py --prompt "Quick question" --json

# Raw output (just the response text, for piping)
python3 scripts/run.py chatgpt.py --prompt "One word: capital of France?" --raw

# Run in background with headless mode (if Cloudflare permits)
python3 scripts/run.py chatgpt.py --prompt "Research task" --headless
```

### Piping to Other Tools
```bash
# Get ChatGPT's response and pipe it
python3 scripts/run.py chatgpt.py --prompt "List 5 trending topics" --raw | head -5

# Use in shell scripts
CHATGPT_RESPONSE=$(python3 scripts/run.py chatgpt.py --prompt "Explain briefly" --raw)
echo "ChatGPT says: $CHATGPT_RESPONSE"
```

## Chat History

### List Recent Conversations
```bash
cd ~/.claude/skills/chatgpt-cli

# List chats with formatted output
python3 scripts/run.py chatgpt.py --list-chats --show-browser

# JSON output with metadata
python3 scripts/run.py chatgpt.py --list-chats --json

# Limit to 10 most recent
python3 scripts/run.py chatgpt.py --list-chats --limit 10

# With debug logging
python3 scripts/run.py chatgpt.py --list-chats --verbose --show-browser
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wolfiesch/chatgpt-cli](https://github.com/wolfiesch/chatgpt-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
