---
trigger: always_on
description: > **Purpose**: Project-specific knowledge for AI coding agents (Antigravity, Cursor, Copilot, etc.)
---

# AGENTS.md — AI Agent Development Guide for GE Demo Generator

> **Purpose**: Project-specific knowledge for AI coding agents (Antigravity, Cursor, Copilot, etc.)
> working on `Code.gs`. This document captures hard-won lessons from production escaping bugs,
> syntax errors, and architectural patterns unique to this codebase.

---

## 1. Architecture: Multi-Layer Code Generation

`Code.gs` is a Google Apps Script (JavaScript) file that **generates bash setup scripts**,
which in turn **generate Python source files** via heredocs. This creates a multi-layer
code generation pipeline where escaping errors are the #1 source of production failures.

```
Layer 1: Code.gs (JavaScript / GAS runtime)
  ↓  JS template literals or string concatenation
Layer 2: Bash setup script (setup-demo-xxx.sh)
  ↓  Heredocs (quoted or unquoted)
Layer 3: Python source files (agent.py, fast_api_app.py, tools.py, etc.)
  ↓  Runtime string operations
Layer 4: LLM system instruction (consumed by Gemini models)
```

### Viewer Escaping Chain (DIFFERENT from above!)

The Data Viewer template (`viewer_app/main.py`) has a **distinct 4-layer chain**
where Layer 3 is a Python triple-quoted string (`"""`) serving HTML, and Layer 4
is browser JavaScript — NOT an LLM instruction.

```
Layer 1: Code.gs (JavaScript / GAS runtime)
  ↓  JS template literal processes \\ → \
  ↓  (note: \n → actual newline!)
Layer 2: Bash quoted heredoc (<<'__VIEWER_MAIN__')
  ↓  Passes through verbatim (no expansion)
Layer 3: Python triple-quoted string (HTML_TEMPLATE = """...""")
  ↓  Python interprets \n → newline, \\ → \
  ↓  (This is the KEY difference from agent heredocs!)
Layer 4: Browser JavaScript execution
  ↓  JS interprets \n in string literals as newline
```

> [!CAUTION]
> The Python `"""` layer is an EXTRA escaping step. In agent heredocs (e.g.,
> `__AGENT_EOF__`), the Python file IS the final destination — Python runtime
> interprets `\n` directly. In the Viewer, Python `"""` renders HTML, and
> the **browser** must see `\n` (literal backslash + n) in the JS source.
> This means you need **4 backslashes** in Code.gs for the Viewer, vs **2**
> for agent heredocs.

### Key File Locations in Code.gs

| Heredoc | Delimiter | Type | Line Range (approx) | Generates |
|---------|-----------|------|---------------------|-----------|
| `.env` | `__ENV_EOF__` | Unquoted | ~L3095 | Runtime environment variables |
| `tools.py` | `__TOOLS_EOF__` | Quoted (`'...'`) | ~L3134-3864 | MCP toolset factories |
| `agent.py` | `__AGENT_EOF__` | Quoted (`'...'`) | ~L5236-5755 | Agent definitions |
| `part_converters.py` | `__PART_CONVERTERS_EOF__` | Quoted | ~L5755-6095 | A2A↔GenAI converters |
| `fast_api_app.py` | `__FAST_API_EOF__` | Quoted (`'...'`) | ~L6107-6993 | A2A server + event loop |
| `viewer_app/main.py` | `__VIEWER_MAIN__` | Quoted (`'...'`) | ~L1841-2420 | Data Viewer Flask app (HTML + JS) |

> [!IMPORTANT]
> Line numbers shift frequently as the file evolves (~8300+ lines). Use `grep` to find
> the actual heredoc boundaries before editing.

---

## 2. Escaping Rules — MANDATORY Reading Before Any Edit

### 2.1 The Golden Rules

1. **NEVER use `\n` in a Python string literal inside a quoted heredoc.** Even though
   `cat <<'EOF'` suppresses shell expansion, the backslash-n sequence (`0x5c 0x6e`)
   causes Python `SyntaxError: unterminated string literal` when the file is written.
   **Use `chr(10)` instead.**

2. **NEVER use f-strings in Python code inside heredocs that also contain JS template
   literal expressions.** The `{variable}` syntax conflicts between Python f-strings
   and JS `${expression}`. **Use string concatenation (`+`) instead.**

3. **NEVER use `$` in Python code inside an UNQUOTED heredoc.** It will be interpreted
   as a shell variable. Quoted heredocs (`'EOF'`) are safe from `$` expansion, but
   unquoted heredocs are not.

4. **Count your backslash layers before writing.** Each layer doubles the backslashes.

5. **For bash line continuation (`\` + newline) in a JS template literal, use `\\` (two
   backslashes), NOT `\\\\` (four).** JS template literal `\\` → single `\` in output →
   valid bash line continuation. Four backslashes produce `\\` in the output, which bash
   interprets as a literal backslash, not a line continuation.

6. **`$()` in a JS template literal does NOT need escaping.** JS only interpolates `${}`.
   `$(command)` is passed through verbatim to the output. Do NOT write `\$()` — that
   produces a literal `\$` in bash, which prevents command substitution.

7. **NEVER use `{variable_name}` or `{{variable_name}}` in agent system instructions
   (base_instruction text).** ADK's `instructions_utils.inject_session_state` uses the
   regex `r'{+[^{}]*}+'` which matches **one or more** opening braces + content + **one
   or more** closing braces. Both `{var}` and `{{var}}` are matched and resolved against
   session state, causing `KeyError` at runtime. **Use `<variable_name>` or
   `[VARIABLE_NAME]` instead.**

8. **NEVER use backtick triplets (` ``` `) anywhere in Code.gs, including inside
   heredoc content, comments, or string literals.** The GAS script editor parses the
   entire file as JavaScript. Backtick triplets are interpreted as JS template literal

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryotat7/ge-demo-generator](https://github.com/ryotat7/ge-demo-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
