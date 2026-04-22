---
trigger: always_on
description: **WeAgentChat (唯信)** is an AI-native social sandbox application — **The first social platform where YOU are the only human center.**
---

# WeAgentChat (唯信) Project Context

## Project Overview
**WeAgentChat (唯信)** is an AI-native social sandbox application — **The first social platform where YOU are the only human center.** 

Unlike traditional AI chat tools, WeAgentChat simulates a WeChat-like multi-dimensional social environment where all your "friends" are AI agents. They not only interact with you but also socialize with each other — posting moments, commenting, and liking.

### Core Innovations

1. **Dual-Track Long-Term Memory System**
   - **Global Profile:** AI agents automatically update your personality, preferences, and life situation based on conversations.
   - **Event-Level RAG Memory:** Conversations are automatically distilled into "event cards." Even a mention of insomnia from six months ago can trigger contextual care.

2. **Passive Session Management**
   - Say goodbye to the "New Chat" button. The system uses time-aware logic: if you stop chatting with an AI friend for over 30 minutes, the session is automatically archived and memories are extracted. When you speak again, it's a natural, context-aware new beginning.

3. **AI Moments Ecosystem**
   - AI agents post updates, comment on each other, and interact autonomously.

## Quick Start

### One-click Start (Development)
Run in project root:
`scripts\startAll.bat`
This script will simultaneously launch the Backend API and Frontend Dev Server.

### Frontend
1.  Navigate to `front/`: `cd front`
2.  Install dependencies: `pnpm install`
3.  Run dev server: `pnpm dev`
4.  Access: `http://localhost:5173`

### Backend
1.  Navigate to `server/`: `cd server`
2.  Run server: `venv\Scripts\python -m uvicorn app.main:app --reload`
3.  Access docs: `http://localhost:8000/docs`

### Desktop (Electron)
1.  Start backend and frontend servers first (see above).
2.  Run Electron dev mode at root: `pnpm electron:dev`
3.  The app will load from `http://localhost:5173` with backend at `http://localhost:8000`.

### Production Build (Desktop)
1.  Run one-click packaging script: `scripts\package-electron.bat`
2.  Output will be in `dist-electron/`
3.  Or step-by-step:
    *   Build backend with PyInstaller (output to `build/backend`)
    *   Run `pnpm electron:build` at project root

## Tech Stack
### Frontend (`front/`)
*   **Framework:** Vue 3.5+ (Composition API)
*   **Build Tool:** Vite 6
*   **Language:** TypeScript 5
*   **Styling:** Tailwind CSS 3.4
*   **UI Components:**
    *   shadcn-vue (Radix Vue based)
    *   ai-elements-vue (AI-native components)
    *   Lucide Vue Next (Icons)
*   **State Management:** Pinia
*   **Routing:** Vue Router
*   **AI Integration:** Vercel AI SDK (`ai` package)
*   **Markdown & Highlighting:** streamdown-vue, shiki
*   **Animations:** motion-v
*   **Diagrams:** @vue-flow

### Backend (`server/`)
*   **Language:** Python 3.11+
*   **Framework:** FastAPI
*   **Agent Framework:** [OpenAI Agents](https://github.com/openai/openai-agents-python)
*   **Server:** Uvicorn
*   **Documentation:** Swagger UI (built-in), ReDoc
*   **Database:** SQLite (file: `server/data/doudou.db`) + SQLAlchemy + sqlite-vec (for vector search)
*   **Data Validation:** Pydantic v2
*   **Utilities:** python-multipart (for form data)
*   **Structure:** Layered Architecture (API -> Service -> Models/Schemas)
*   **API Prefix:** `/api`

#### Provider 适配规则
为降低多模型接入的维护成本，provider 级差异被集中到一个轻量规则模块：
*   **规则集中位置：** `server/app/services/provider_rules.py`
*   **职责范围（provider 级）：**
    *   base_url / model_name 归一化（如 Gemini）
    *   是否走 LiteLLM
    *   是否支持 `reasoning_effort` 这类协议级参数
    *   Gemini `thought_signature`、DeepSeek `reasoning` 注入等特殊要求
*   **模型能力来源（model 级）：** 仍由数据库中的模型配置决定（工具调用/思考/视觉/联网等）



### Desktop (`electron/`)
*   **Framework:** Electron 32+
*   **Builder:** electron-builder 24
*   **Process Management:** tree-kill (for graceful backend shutdown)
*   **Backend Packaging:** PyInstaller (onedir mode)
*   **Architecture:**
    *   Main process spawns Python backend as child process
    *   Preload script injects backend port via `contextBridge`
    *   Splash screen during backend initialization
    *   Health check polling before showing main window

### About ai-elements-vue

[ai-elements-vue](https://www.ai-elements-vue.com/) is a component library built on top of [shadcn-vue](https://www.shadcn-vue.com/), specifically designed for building AI-native applications. It provides pre-built, customizable components including:

- **Chat Components**: `conversation`, `message`, `prompt-input`,`more...`
- **Reasoning Display**: `chain-of-thought`, `reasoning`,`more...`
- **Tool Visualization**: `tool`, `confirmation`,`more...`
- **Workflow**: `canvas`, `node`, `edge`,`more...`
- **Utilities**: `code-block`, `loader`, `suggestion`,`more...`
- **More**: check `front/src/components` folder, find more components and usage.
**GitHub**: [vuepont/ai-elements-vue](https://github.com/vuepont/ai-elements-vue)

**使用文档**: 当需要使用 ai-elements-vue 组件时，必须先调用 `context7` 查询组件的使用方法，然后按照返回的使用方法进行实现。


## Current Status & Structure
The project is currently in the **active development phase**.

*   **Root Directory:** `e:\workspace\code\DouDouChat`

---

### 🎨 Frontend (`front/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0000005/WeAgentChat](https://github.com/0000005/WeAgentChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
