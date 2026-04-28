---
trigger: always_on
description: **Version: v0.1.0** — submitted to Chrome Web Store for review.
---

# CLAUDE.md — AgentGrow Chrome Extension

**Version: v0.1.0** — submitted to Chrome Web Store for review.

Open-source AI browser assistant that **automates common browser tasks** — filling forms, writing email drafts, creating/editing/summarizing documents, navigating complex sites — to save users time. Connects to **any LLM provider** (OpenRouter, OpenAI, Anthropic, Groq, Ollama, or any OpenAI-compatible endpoint) via user-supplied API URL + key. No subscriptions. No vendor lock-in.

- **Extension homepage:** https://devops.gheware.com/agentgrow/
- **GitHub:** https://github.com/brainupgrade-in/agentgrow-chrome-extension

**Full design (architecture, data models, streaming, reliability, test suite):** `agentgrow.io-chrome-extension-design.md`  
**Reference studies:** `abacusai-chrome-ext.md`, `chrome-extensions-dev-best-practices.md`

---

## Project Goal

Build a Chrome extension (Manifest V3) that helps users automate repetitive browser tasks via AI — comparable to AbacusAI and Claude's browser extensions, but fully open source and LLM-provider-agnostic. Key differentiator: **live DOM read/write** on the adjacent tab — no screenshots, no snapshots.

### Core Use Cases
- **Form Automation** — auto-fill job applications, registrations, checkout forms, surveys
- **Email & Message Drafting** — compose emails in Gmail/Outlook, messages in Telegram/Slack
- **Document Tasks** — summarize articles, extract data (emails, links, tables, prices), create drafts
- **Site Navigation** — find information on complex sites, answer questions about page content
- **Data Extraction** — pull structured data from any page into usable formats

### Implemented Features (v0.1.0)
1. **Side panel chat with streaming** — persistent AI chat alongside any webpage; SSE streaming (OpenAI-compatible providers) + NDJSON streaming (Ollama) via named Port `llm-stream`
2. **Conversation persistence** — active conversation saved to `chrome.storage.local`, restored on panel reopen
3. **Smart auto-context** — auto-reads page content + text selection automatically; no manual toggles needed
4. **Multi-model dropdown** — switch models in 1 click; shows all preset models per provider + dynamically fetched models from any endpoint
5. **Chat UX** — New Chat button, copy message to clipboard, retry failed messages, timestamps on hover
6. **DOM write** — form fill (React/Angular/Vue compatible via native setter + `InputEvent` + full event suite), contenteditable support (Telegram, Slack, Gmail, Notion), text highlight, cursor insert
7. **DOM click & navigate** — click any button/link by CSS selector, scrollIntoView, React-compatible MouseEvent dispatch
8. **Smart selector resolution** — 5-level fallback: id → name → aria-label → placeholder → positional
9. **Action safety mode** — "Ask before acting" (default, shows approve/cancel UI) vs "Auto-act" with persistent risk warning
10. **Provider management** — add/edit/delete any OpenAI-compatible endpoint or Ollama; 7 built-in presets (OpenRouter, OpenAI, Anthropic, Groq, Google Gemini, Ollama, Custom); save confirmation toast; provider selection persisted to `chrome.storage.local`
11. **Ollama Cloud support** — api.ollama.com with API key; auto-shows key field for non-localhost Ollama URLs
12. **Private network support** — test connection fallback via tab execution for 192.168.*/10.*/172.16.* servers (bypasses service worker fetch restrictions)
13. **Friendly error messages** — 401 → "Invalid API key", 429 → "Rate limited", 404 → "Model not found", 5xx → "Server error"; retry button on all errors
14. **Error boundary** — wraps entire side panel; React crashes show recovery UI instead of blank screen
15. **Page reader** — extracts fillable fields + clickable elements with verified CSS selectors, contenteditable detection, UI noise filtering, line deduplication
16. **Dynamic model discovery** — "Fetch models from endpoint" button in provider form; fetches `/v1/models` (OpenAI-compatible) or `/api/tags` (Ollama); cached per-provider; chat header dropdown merges preset + fetched models; works with private network fallback
17. **In-page activity toast** — shadow-DOM toast at bottom of active tab showing "AgentGrow is working/filling/clicking/typing on this tab" with pulsing indicator + **Stop** button; appears during LLM streaming and DOM write operations; Stop aborts the stream and clears all toasts

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Build | Vite + `@crxjs/vite-plugin` |
| Language | TypeScript 5.x (strict) |
| UI | React 18 |
| State | Zustand (slice-per-domain) |
| Styling | Tailwind CSS + CSS variables |
| Markdown | react-markdown + remark-gfm + **rehype-sanitize** |
| Icons | Lucide React |
| Validation | **Zod** (all message payloads + provider URL validation) |
| Tests (unit) | Vitest + jsdom |
| Tests (e2e) | Playwright |
| API mocking | MSW (Mock Service Worker) |
| Crypto | Web Crypto API (built-in, no deps) |
| Package manager | pnpm |

---

## Chrome Extension Architecture (MV3)

```
Side Panel (React)  ←──sendMessage──→  Service Worker  ←─executeScript──→  Page DOM (reads)
Side Panel (React)  ←═══ Port ════════ Service Worker  ←──sendMessage──→  Content Script (writes)
                                             ↕

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brainupgrade-in) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
