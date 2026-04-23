---
trigger: always_on
description: **Generated:** 2026-04-10
---

# OpenBrowser Project Knowledge Base

**Generated:** 2026-04-10
**Commit:** 25b3a2e (main)
**Stack:** Python 3.12+ (FastAPI) + TypeScript (Chrome Extension MV3)

## OVERVIEW

Visual AI assistant for browser automation powered by Qwen3.5-Plus (primary) with Qwen3.5-Flash support as a cost-effective alternative. Provides AI-powered visual understanding and interaction for web automation, data extraction, interactive workflows, and a record -> compile -> replay pipeline for reusable Browser Routines. Single-model automation loop: visual perception → decision making → browser interaction → verification.

## STRUCTURE

```
OpenBrowser/
├── server/           # FastAPI backend + agent logic + WebSocket
│   ├── agent/        # Agent orchestration and tool definitions
│   ├── api/          # REST endpoints
│   ├── core/         # Core processing logic
│   └── websocket/    # WebSocket server
├── extension/        # Chrome extension (MV3) for browser control
├── frontend/         # Static web UI (HTML)
├── eval/             # Mock sites + routine compile/replay evaluation
└── reference/        # External SDK references (read-only)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Agent orchestration | `server/agent/manager.py` | Conversation lifecycle, LLM config |
| Browser commands | `server/core/processor.py` | Command routing, multi-session |
| Dialog handling | `server/models/commands.py` | HandleDialogCommand, DialogAction |
| REST API routes | `server/api/routes/` | FastAPI endpoints |
| Recording routes | `server/api/routes/recordings.py` | Recording lifecycle, workflow draft, compiler, finalize |
| Routine routes | `server/api/routes/routines.py` | Saved Browser Routine CRUD for replay |
| Browser UUID routing | `server/api/routes/browsers.py` | Browser UUID registration and validation |
| WebSocket handling | `server/websocket/manager.py` | Extension communication |
| Browser UUID registry | `server/core/uuid_manager.py` | `uuid -> websocket` capability mapping |
| Command models | `server/models/commands.py` | Pydantic command/response types |
| Recording persistence | `server/core/recording_manager.py` | SQLite recording sessions/events, immutability boundaries |
| Workflow draft compiler | `server/core/workflow_compiler.py` | Normalize raw recording traces into high-level draft steps/IR |
| Compiler Agent | `server/core/compiler_agent.py` | TraceViewer, clarify-with-user loop, Routine validation |
| Routine persistence | `server/core/routine_manager.py` | Saved routines linked back to source recordings |
| **Prompt templates** | `server/agent/prompts/` | **Jinja2 templates for agent prompts** |
| Tab tool | `server/agent/tools/tab_tool.py` | TabTool for tab management |
| Highlight tool | `server/agent/tools/highlight_tool.py` | HighlightTool for element discovery |
| Element interaction | `server/agent/tools/element_interaction_tool.py` | ElementInteractionTool with 2PC flow |
| Dialog tool | `server/agent/tools/dialog_tool.py` | DialogTool for dialog handling |
| ToolSet aggregator | `server/agent/tools/toolset.py` | OpenBrowserToolSet aggregates all 4 tools |
| Extension entry | `extension/src/background/index.ts` | Command handler, dialog processing |
| Extension recorder | `extension/src/recording/recorder.ts` | Recording scope, event capture, keyframe upload |
| Recording keyframe policy | `extension/src/recording/keyframe-policy.ts` | Which events get screenshots and when drift is discarded |
| Dialog manager | `extension/src/commands/dialog.ts` | CDP dialog events, cascading |
| JavaScript execution | `extension/src/commands/javascript.ts` | CDP Runtime.evaluate, dialog race |
| Screenshot capture | `extension/src/commands/screenshot.ts` | CDP Page.captureScreenshot |
| Tab management | `extension/src/commands/tab-manager.ts` | Session isolation, tab groups |
| UUID page | `extension/src/uuid/uuidPage.ts` | Browser UUID display and registration status |
| Frontend recording/replay UI | `frontend/index.html` | Browser UUID input, recording panel, compile flow, saved routines, slash-menu replay |
| Routine evaluation | `eval/routine_eval/` | Compile-track + replay-track eval harness for record/replay |

## ARCHITECTURE

```
┌─────────────────────────────────────────┐
│       Qwen3.5 Family (Multimodal LLM)   │
│   Qwen3.5-Plus (primary) / Flash (cost-effective)
│   Visual Perception │ Decision Making │ Browser Control
└────────────────────┬────────────────────┘
                     │
┌────────────────────▼────────────────────┐
│   OpenBrowser Agent Server (FastAPI)    │
│   - REST API (port 8765)                │
│   - WebSocket (port 8766)               │
│   - Session Management                   │
│   - Tool Orchestration                   │
└────────────────────┬────────────────────┘
                     │
┌────────────────────▼────────────────────┐
│   Chrome Extension (CDP)                │
│   - JavaScript execution (with race)    │
│   - Dialog detection & handling         │
│   - Screenshots (1280x720)              │
│   - Tab management with groups          │
└─────────────────────────────────────────┘
```

## BROWSER UUID AUTHORIZATION

OpenBrowser now uses the browser UUID as a capability token, not just an internal identifier.

### Permission Model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [softpudding/OpenBrowser](https://github.com/softpudding/OpenBrowser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
