---
trigger: always_on
description: Unified LLM proxy for AI agents. Route all model requests through http://localhost:18080/v1. Provides FREE access to Gemini CLI, Claude CLI, Codex, and Vibe via your existing subscriptions. Includes Intelligent Auto-Routing (Cortex) for autonomous model selection, a self-optimizing Lab, token conservation, and a management dashboard. Use when: (1) making LLM calls, (2) switching between CLI/Local/Cloud providers, (3) attaching local files to prompts, (4) needing intelligent routing, or (5) auto-
---


# switchAILocal Skill

Unified LLM proxy and intelligent router for AI agents. Always use `http://localhost:18080/v1` as your base URL.

**The killer feature**: Use your paid CLI subscriptions (Gemini Pro, Claude Pro, etc.) via the API — **it's FREE** because you already pay for the subscription!

## When to Use

✅ **USE this skill when:**
- An agent needs to make an LLM call to Gemini, Claude, Ollama, OpenAI, etc.
- You need to attach entire file directories to an LLM context (via `geminicli` and `claudecli`).
- You need to start, stop, or manage the local switchAILocal server/bridge.
- You want to monitor provider health, analytics, or status.
- You want to auto-detect available models and configure optimal routing.

❌ **DON'T use this skill when:**
- The user asks about their calendar, emails, or non-LLM operations.
- Interacting with APIs that aren't OpenAI-compatible.

---

## 🤖 Agent Auto-Setup (Zero-Config Bootstrap)

Follow these steps IN ORDER to bootstrap switchAILocal from scratch on a user's machine.

### Step 1 — Install & Start

**Option A (npx — recommended, no git clone needed):**
```bash
npx @traylinx/switchailocal
```

**Option B (from source):**
```bash
git clone https://github.com/traylinx/switchAILocal.git
cd switchAILocal
./ail.sh setup
source ~/.zshrc 2>/dev/null || source ~/.bashrc 2>/dev/null  # Reload PATH
```

### Step 2 — Detect Available Providers

Run these commands to discover what the user has installed:

```bash
# CLI Providers (FREE!)
which gemini  && echo "✅ Gemini CLI found"
which claude  && echo "✅ Claude CLI found"
which codex   && echo "✅ Codex CLI found"

# Local Providers
curl -s http://localhost:11434/api/tags 2>/dev/null && echo "✅ Ollama running"

# Cloud API Keys (check environment)
[ -n "$OPENAI_API_KEY" ]    && echo "✅ OpenAI key found"
[ -n "$ANTHROPIC_API_KEY" ] && echo "✅ Anthropic key found"
[ -n "$GEMINI_API_KEY" ]    && echo "✅ Google AI key found"
```

### Step 3 — Generate config.yaml

Based on detected providers, generate a minimal config:

```yaml
host: ""
port: 18080

# Enable any detected CLI providers:
# geminicli: (uses `gemini` CLI — FREE with Google AI Premium)
# claudecli: (uses `claude` CLI — FREE with Claude Pro)
# codex: (uses `codex` CLI — FREE with OpenAI Plus)

# Enable Ollama if detected:
ollama:
  enabled: true
  base-url: "http://localhost:11434"
  auto-discover: true

# Enable Intelligent Auto-Routing:
auto-routing:
  enabled: true
  weights:
    availability: 0.35
    quota: 0.25
    latency: 0.2
    success-rate: 0.2
  discovery:
    enabled: true
    probe-on-startup: true
  conservation:
    enabled: true
    simple-threshold-tokens: 500
  lab:
    enabled: true
    adaptation-interval: 24h
    max-weight-drift: 0.1
```

### Step 4 — Start & Verify

```bash
ail start
# Verify it's running:
curl -s http://localhost:18080/v1/models | head -c 200
```

You should see a JSON response listing all available models/providers.

---

## ⚠️ Critical: Model Format

**NEVER use bare model names.** Format is ALWAYS `provider:` or `provider:model`.

| ❌ Wrong             | ✅ Correct                  | Why                       |
| ------------------- | -------------------------- | ------------------------- |
| `gemini-2.5-pro`    | `geminicli:gemini-2.5-pro` | Needs provider prefix     |
| `claude-3-5-sonnet` | `claudecli:`               | `claudecli:` uses default |
| `llama3`            | `ollama:llama3`            | Needs provider prefix     |
| `auto route me`     | `auto` or `auto:coding`    | Use `auto` prefix only    |

---

## 🏗️ Provider Reference

### 1. CLI Providers (FREE!)
Uses your human's CLI subscriptions. Best for agents.

| Prefix       | CLI      | Subscription Required |
| ------------ | -------- | --------------------- |
| `geminicli:` | `gemini` | Google AI Premium/Pro |
| `claudecli:` | `claude` | Claude Pro/Max        |
| `codex:`     | `codex`  | OpenAI Plus           |
| `vibe:`      | `vibe`   | Mistral Le Chat       |

### 2. Local & Cloud

| Prefix      | Source         | Cost                   |
| ----------- | -------------- | ---------------------- |
| `ollama:`   | Local Ollama   | FREE                   |
| `auto`      | Cortex Router  | FREE (auto-selects)    |
| `switchai:` | Traylinx Cloud | Per-token              |

### 3. switchAI Cloud Aliases

| Alias              | Upstream Model        | Best For      |
| ------------------ | --------------------- | ------------- |
| `switchai-fast`    | `openai/gpt-oss-20b`  | Fast tasks    |
| `switchai-chat`    | `openai/gpt-oss-20b`  | Conversation  |
| `switchai-reasoner`| `deepseek-reasoner`   | Deep thinking |

---

## 🧠 Intelligent Auto-Routing (Cortex)

When the model is `auto` or `auto:<intent>`, the Cortex Router automatically selects the best available model using a composite scoring algorithm.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [traylinx/switchAILocal](https://github.com/traylinx/switchAILocal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
