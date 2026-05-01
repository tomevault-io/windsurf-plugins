---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mysti is a VSCode extension providing a unified AI coding assistant interface supporting 12 AI backends (Claude Code, OpenAI Codex, Google Gemini, Cline, GitHub Copilot, Cursor, OpenClaw, OpenCode, Qwen Code, Ollama, LocalAI, and Manus). It features sidebar/tab chat panels, conversation persistence, multi-agent brainstorm mode (any 2 of 11 agents with 5 collaboration strategies), autonomous mode with safety classification, @-mention agent routing, permission controls, plan selection, context compaction, and a three-tier agent loading system for personas and skills.

## Build Commands

```bash
npm run compile           # Production build (webpack)
npm run watch             # Development build with watch mode
npm run lint              # ESLint check (src/**/*.ts)
npm run sync-agents       # Sync plugins from wshobson/agents repo
npm run sync-agents:force # Force sync (ignores 24h cache)
npx vsce package          # Package extension as .vsix
```

Output: `dist/extension.js` from entry point `src/extension.ts` (webpack bundles with ts-loader, target: node, CommonJS2).

**Note:** Tests are not yet implemented (`npm run test` exists but has no test files).

## Development

Press `F5` in VSCode to launch Extension Development Host for debugging. Set breakpoints in TypeScript files and filter Debug Console with `[Mysti]` for extension logs.

**CLI requirements**: At least one of these CLIs must be installed for the extension to function:

- `npm install -g @anthropic-ai/claude-code` (Claude Code)
- `npm install -g @google/gemini-cli` (Gemini)
- `npm install -g @github/copilot-cli` (GitHub Copilot)
- Codex CLI (OpenAI - follow their installation guide)
- `npm install -g cline` (Cline)
- `curl https://cursor.com/install -fsS | bash` (Cursor)
- `npm install -g openclaw@latest` (OpenClaw)
- `npm i -g opencode-ai@latest` (OpenCode)
- `npm install -g @qwen-code/qwen-code@latest` (Qwen Code)
- Ollama (install from ollama.com)
- LocalAI (install from localai.io)

## Architecture

### Core Pattern: Manager + Provider Facades

```
extension.ts (entry — activate() wires everything)
    │
    ├── Managers (business logic, src/managers/)
    │   ├── ContextManager        - File/selection tracking (per-panel contexts)
    │   ├── ConversationManager   - Message persistence via globalState
    │   ├── ProviderManager       - Provider registry facade
    │   ├── PermissionManager     - Access control
    │   ├── BrainstormManager     - Multi-agent orchestration (5 strategies)
    │   ├── ResponseClassifier    - AI-powered response analysis
    │   ├── PlanOptionManager     - Implementation plan extraction
    │   ├── SuggestionManager     - Quick action suggestions
    │   ├── SetupManager          - CLI auto-setup & authentication
    │   ├── AgentLoader           - Three-tier agent loading from markdown
    │   ├── AgentContextManager   - Recommendations & prompt building
    │   ├── TelemetryManager      - Anonymous usage analytics
    │   ├── AutocompleteManager   - Autocomplete functionality
    │   ├── AutonomousManager     - Semi/full autonomous mode orchestration
    │   ├── MemoryManager         - Learning memory for autonomous preferences
    │   ├── SafetyClassifier      - Three-level safety evaluation (safe/caution/blocked)
    │   ├── CompactionManager     - Context overflow prevention
    │   ├── MentionRouter         - @-mention routing to specific agents
    │   ├── SlashCommandManager   - Unified slash command menu system
    │   ├── AgentLifecycleManager - Session idle timeout & child process tracking
    │   ├── ActiveModeManager     - OpenClaw daemon WebSocket connection
    │   └── ChannelBridge         - Routes messages between daemon channels and panels
    │
    └── ChatViewProvider (UI coordinator, src/providers/ChatViewProvider.ts)
            │
            ├── Webview UI (src/webview/webviewContent.ts — embedded HTML/CSS/JS)
            │
            └── Providers (CLI integrations, src/providers/<name>/)
                ├── ClaudeCodeProvider  (extends BaseCliProvider)
                ├── CodexProvider       (extends BaseCliProvider)
                ├── GeminiProvider      (extends BaseCliProvider)
                ├── ClineProvider       (extends BaseCliProvider)
                ├── CopilotProvider     (extends BaseCliProvider)
                ├── CursorProvider      (extends BaseCliProvider)
                ├── OpenClawProvider    (extends BaseCliProvider + Gateway WebSocket)
                ├── OpenCodeProvider    (extends BaseCliProvider)
                ├── QwenCodeProvider    (extends BaseCliProvider)
                ├── OllamaProvider      (extends BaseCliProvider)
                ├── LocalAIProvider     (extends BaseCliProvider)
                └── ManusProvider       (extends BaseCliProvider, API-based)
```

### Key Design Decisions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeepMyst/Mysti](https://github.com/DeepMyst/Mysti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
