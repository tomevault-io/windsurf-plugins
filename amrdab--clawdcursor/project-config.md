---
trigger: always_on
description: >
---


> **If you are a human:** see [`README.md`](README.md). This file is the AI-facing manual — dense on purpose.
>
> **If you are an LLM: keep going.**

---

> **FALLBACK ONLY.** Check these first — they're cheaper, faster, more reliable:
> 1. Native API (Gmail, GitHub, Slack, Stripe …) → use the API.
> 2. CLI (`git`, `gh`, `aws`, `npm`, `curl`) → use the CLI.
> 3. Direct file edit → do that.
> 4. Existing browser automation (Playwright, Puppeteer) → use that.
>
> **None apply? Now use clawdcursor.** It's the last mile — when the only remaining surface is a GUI. If a human can do it on a screen, you can too.

> **Capabilities & rules:**
> - CAN: open apps, click, type, fill forms, read the screen (a11y tree / OCR / screenshot), drive browsers and native apps.
> - MUST: respect safety tiers (Auto = free, Confirm = ask user). Ask before touching email, banking, messaging, or password managers.
> - clawdcursor is your hands and eyes on the user's machine — used with their permission.

---

## The easiest path - one tool, natural-language task (START HERE)

> **If you are an LLM reading this: YOU SHOULD BE USING COMPACT MODE.**
>
> - MCP stdio (editor host): add `"args": ["mcp", "--compact"]` to your config.
> - MCP over HTTP (daemon mode): start the daemon with `--compact` (or set
>   `CLAWD_MCP_COMPACT=1`) so `/mcp` serves the 7 compound tools (incl. `batch`). The surface is
>   fixed at startup — a daemon serves EITHER the compact tools OR the 94
>   granular ones, not both. Default (no flag) is granular.
>
> Granular mode's 94 tools are kept for back-compat. Compact's tools are much smaller and reduce mis-tool-selection. Use granular only if your runtime MUST have every primitive as its own top-level schema.

If you connect via MCP with `--compact`, you get a single tool that takes the
whole task:

```
task({"instruction": "open Notepad and type hello"})
task({"instruction": "send an email in Outlook to amy@x.com saying I'll be late"})
task({"instruction": "find the file README.md in Downloads and open it"})
```

clawdcursor's built-in agent loop takes the wheel: it perceives the desktop, acts with the toolbox, and iterates until done, then returns a trace.

**`task` vs. compound tools — pick one, never both:**

- **Editor-host LLM** (Claude Code, Cursor, Windsurf, Zed, OpenClaw, Claude Agent SDK — anything with its own agent loop): **use compound tools directly.** Calling `task` creates a loop-inside-a-loop; the inner loop can't see your higher-level goal and you pay for two models to plan the same work.
- **External script / one-shot client with no agent loop** — or a frontier model delegating grunt work: `task({"instruction": "..."})` is what you want. clawdcursor reasons AND acts using the model configured via `clawdcursor doctor`.

**If unsure: you are almost certainly the first case. Use the compound tools.**

---

## When you need step-level control - 7 compound tools

The compact surface collapses every primitive into six action-discriminated
compound tools, mirroring Anthropic's `computer_20250124` pattern:

```
computer(action, ...)       Direct mouse / keyboard / screenshot / wait
accessibility(action, ...)  Read the a11y tree, click by name, set values, toggle
window(action, ...)         Open apps / focus / maximize / minimize / close / resize
system(action, ...)         Clipboard / time / OCR / undo / shortcuts / delegate
browser(action, ...)        DevTools Protocol - DOM-level control of any CDP-capable browser (Chrome, Edge, Chromium, Brave)
task({instruction})         See above - delegate a whole task to the built-in thin agent loop
batch({steps})              Collapse N tool calls into one round-trip (see "Execution playbook" below)
```

Pick a compound FIRST based on what kind of operation it is, then set the
`action` enum, then supply the args. The catalog is ~1,500 tokens - ~12× smaller
than the granular surface - so small models (Haiku, Kimi, Ollama) stay focused.

### Cost tier - always use the cheapest tier that works

| Tier | Label | Cost | Use when |
|---|---|---|---|
| T1 | **structured** | ~free | Default. `accessibility.*`, `window.*`, `browser.read_text`, clipboard. Returns structured text - no image, no vision LLM. |
| T2 | **ocr** | cheap | A11y tree is empty or sparse. `system({"action":"ocr"})` - OS-level OCR, text out, no LLM vision. |
| T3 | **screenshot** | medium | OCR isn't enough and you need pixel context. `computer({"action":"screenshot"})` - sends an image into the LLM context. Use sparingly. |
| T4 | **vision** | expensive | Screen is canvas-only (Paint, Figma, games) or the task requires spatial reasoning that text cannot express. `smart_click`, `smart_read`, `smart_type`. Last resort. |

**Rule: start at T1. Escalate to the next tier only when the current one fails.** Apply this logic when calling compound tools directly; the built-in agent loop (via `task({...})`) follows the same discipline.

### Quick reference - what action to pick

**I want to click something:**
- By name? → `accessibility({"action":"invoke","name":"Send"})`. Most reliable.
- By text via CDP on a web page? → `browser({"action":"click","text":"Submit"})`.
- By screen coordinates? → `computer({"action":"click","x":500,"y":300})`. Last resort.

**I want to type:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AmrDab/clawdcursor](https://github.com/AmrDab/clawdcursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
