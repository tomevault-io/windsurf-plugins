---
trigger: always_on
description: Integrate SigMap with OpenCode, OpenHands, Cline, Aider, and local LLMs via Ollama, llama.cpp, vLLM. Model-agnostic context for any inference backend.
---


# Open-source agents and local LLM workflows

SigMap is **model-agnostic** — it works with any AI assistant or inference backend that consumes markdown context files. Whether you're using a proprietary cloud API or running an open-source model locally, SigMap handles context generation the same way.

This guide shows how to integrate SigMap with popular open-source coding agents and local inference backends.

## Model-agnostic context generation

SigMap produces plain markdown files containing your codebase signatures and context. Any tool that reads markdown can use SigMap output:

```bash
sigmap  # generates .github/copilot-instructions.md
```

Your AI agent reads this file, either by:
1. **Manual paste** — copy the markdown into your chat
2. **File watcher** — auto-reload when you run `sigmap --watch`
3. **IDE integration** — MCP, .cursor/mcp.json, or Claude Code settings
4. **API integration** — HTTP fetch from a local endpoint
5. **CLI pipe** — direct stdout stream to your model

---

## Open-source coding agents

### OpenCode

**Status:** ⭐ **Most popular** (157k GitHub stars)  
**Type:** Open-source coding agent  
**Model:** Works with OpenAI API (default) or OpenAI-compatible servers (Ollama, local vLLM)

OpenCode is the most widely-adopted open-source coding agent in the LocalLLM community. It integrates with SigMap via file context injection.

#### Setup with SigMap

1. **Generate base context**
   ```bash
   sigmap
   # Writes: .github/copilot-instructions.md
   ```

2. **Start OpenCode**
   ```bash
   # With cloud LLM (OpenAI, Anthropic, etc.)
   opencode --model gpt-4

   # With local inference (see "Local LLM inference" section below)
   opencode --api-base http://localhost:8000 --model local-model
   ```

3. **Inject context in OpenCode**  
   When OpenCode opens the file editor, paste the contents of `.github/copilot-instructions.md` at the top of your current file as a comment block:
   
   ```javascript
   // === SigMap context (paste from .github/copilot-instructions.md) ===
   // ## File signatures
   // auth/login.js: login(email, password) → Promise<{token, user}>
   // auth/verify.js: verify(token) → boolean
   // ... (rest of context)
   // ===
   
   // Your actual code here
   ```

4. **Auto-refresh context during active development**  
   Keep OpenCode running while you code:
   ```bash
   sigmap --watch
   ```
   OpenCode will see the updated `.github/copilot-instructions.md` when you reload the editor.

#### Integration pattern

OpenCode's strength is **local development with full IDE awareness**. Use SigMap to pre-select relevant files before asking:

```bash
# Before asking OpenCode about auth, rank the files
sigmap ask "How is authentication handled?" --top 10
# Copy those file signatures into the context

# Then ask OpenCode: "Given the file signatures, explain the auth flow"
```

---

### OpenHands

**Status:** ⭐ **Growing** (75k GitHub stars)  
**Type:** Open-source autonomous agent  
**Model:** Works with any OpenAI-compatible API

OpenHands runs as a web interface and can be configured to read codebase context.

#### Setup with SigMap

1. **Generate context**
   ```bash
   sigmap --json > /tmp/sigmap-context.json
   ```

2. **Start OpenHands with context path**
   ```bash
   CONTEXT_FILE=/path/to/.github/copilot-instructions.md openhands
   ```

3. **Use SigMap in prompts**  
   In the OpenHands chat, reference the context:
   ```
   Review the files in .github/copilot-instructions.md and explain the auth system.
   ```

---

### Cline / Roo Code

**Status:** ⭐ **Popular** (61k GitHub stars)  
**Type:** Open-source coding agent for VS Code/Cursor  
**Model:** Works with OpenAI API or local models (via OpenAI-compatible Base URL)

Cline and Roo Code are VSCode/Cursor extensions that provide agent-like coding assistance.

#### Setup with SigMap

1. **Install Cline or Roo Code**
   ```bash
   # In VS Code: Install from Extensions → search "Cline" or "Roo Code"
   ```

2. **Configure to use SigMap context**  
   In your Cline settings (`.cline.md` in project root):
   ```bash
   # Auto-include SigMap context
   sigmap
   ```

3. **Use in Cline prompts**  
   Start your Cline request with:
   ```
   Read .github/copilot-instructions.md as project context, then implement X.
   ```

---

### Aider

**Status:** ⭐ **Established** (41k GitHub stars)  
**Type:** Open-source AI pair programmer (CLI)  
**Model:** Works with OpenAI API or local models

Aider is a terminal-based AI pair programmer that can reference external context files.

#### Setup with SigMap

1. **Generate context**
   ```bash
   sigmap
   ```

2. **Add SigMap output to Aider's context**
   ```bash
   # Copy the context file to Aider's awareness
   cp .github/copilot-instructions.md .aider.context.md
   ```

3. **Use Aider with context**
   ```bash
   aider --file src/auth.js \
     --read .aider.context.md \
     "Implement the login handler using the context provided"
   ```

---

## Local LLM inference backends

These are **inference engines**, not coding agents. They run the actual LLM model. Pair them with an agent (Cline, OpenCode, Aider) above using the "OpenAI-compatible Base URL" pattern (see next section).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manojmallick/sigmap](https://github.com/manojmallick/sigmap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
