---
trigger: always_on
description: 始终用中文回复，包含生成的文件内容以及对话框的对话，除非是专有名词、公式以及代码
---


---
trigger: always_on
---

# cursor.md

始终用中文回复，包含生成的文件内容以及对话框的对话，除非是专有名词、公式以及代码

This file provides guidance to Cursor  when working with code in this repository.

(cursor.md holds the rules for the Cursor project, while CLAUDE.md is for Claude Code. As Cursor, please refer only to cursor.md. Both files contain the exact same information.)

## Project Overview

**TruthSeeker** is a cross-modal malicious AIGC detection system for the CISCN2026 competition. It uses a multi-agent architecture with LangGraph orchestration, featuring four specialized agents (Forensics, OSINT, Challenger, Commander) that debate and converge on a final verdict.

### High-Level Architecture

```
Frontend (Next.js15 + React19)
├──3D Bento Box UI (React Three Fiber v9)
├── Real-time Agent State Visualization
└── Expert Collaboration Mode

Backend (FastAPI + LangGraph v1.0+)
├── LangGraph State Machine (TypedDict-based)
├── Four-Agent Debate System
│ ├── Forensics Agent (Audio/Video analysis)
│ ├── OSINT Agent (Threat intelligence)
│ ├── Challenger Agent (Logic verification)
│ └── Commander Agent (Final verdict)
└── SSE Streaming for Real-time Updates

Database (Supabase)
├── PostgreSQL with RLS
├── Realtime Broadcast/Presence
└── Vector storage for embeddings
```

## Technology Stack Lock

**Version freeze as of2026-03-01 - DO NOT upgrade without verification:**

### Frontend
- Next.js ^15.2.0 with App Router
- React ^19.0.0
- Tailwind CSS ^4.0.0 (CSS-first config, no tailwind.config.js)
- shadcn/ui @canary (for Tailwind v4 support)
- **motion** ^12.9.2 (formerly framer-motion, import from "motion/react")
- @react-three/fiber ^9.5.0 + @react-three/drei ^10.0.0
- @supabase/ssr ^0.5.0 (auth-helpers is deprecated)

### Backend
- FastAPI0.134.0
- LangGraph >=1.0.9 (CRITICAL: State must use TypedDict, NOT Pydantic)
- Python ^3.11

Never skip ahead - each layer has defined deliverables and milestones.

## Reference Documents (MUST READ before coding)

The following documents in `/docs` directory contain authoritative specifications. **Always read the relevant document before implementing any feature:**

| Document | Path | Purpose | When to Read |
|----------|------|---------|--------------|
| **PRD.md** | `docs/PRD.md` | Product requirements, core business logic, agent definitions, competition scenarios | Before ANY implementation to understand requirements |
| **TECH_STACK.md** | `docs/TECH_STACK.md` | Version-locked dependencies, initialization commands, breaking changes | Before installing dependencies or upgrading packages |
| **IMPLEMENTATION_PLAN.md** | `docs/IMPLEMENTATION_PLAN.md` |60-day roadmap, daily tasks, milestone checkpoints | At the start of each development day |
| **FRONTEND_GUIDELINES.md** | `docs/FRONTEND_GUIDELINES.md` | Next.js15 patterns, Tailwind v4 config, motion imports, R3F setup | Before writing frontend code |
| **BACKEND_STRUCTURE.md** | `docs/BACKEND_STRUCTURE.md` | LangGraph v1.0+ patterns, TypedDict State, agent architecture, SSE implementation | Before writing backend code |
| **APP_FLOW.md** | `docs/APP_FLOW.md` | User journey, page flows, state transitions, Realtime events | Before designing UI interactions or API contracts |

### Development Workflow

1. **Before starting a task**: Read IMPLEMENTATION_PLAN.md to confirm current phase and priorities
2. **Before frontend coding**: Read FRONTEND_GUIDELINES.md + TECH_STACK.md (check versions)
3. **Before backend coding**: Read BACKEND_STRUCTURE.md + TECH_STACK.md (check LangGraph rules)
4. **Before UI design**: Read APP_FLOW.md + PRD.md (section3.1 Frontend modules)
5. **Before adding dependencies**: Read TECH_STACK.md breaking changes section

### Critical Cross-References

- **Agent State structure**: PRD.md (section2.1) + BACKEND_STRUCTURE.md
- **Color scheme/Tokens**: PRD.md (section3.1) + FRONTEND_GUIDELINES.md
- **API Contracts**: APP_FLOW.md (state transitions) + BACKEND_STRUCTURE.md (endpoints)
- **Database Schema**: BACKEND_STRUCTURE.md + PRD.md (evidence board requirements)

## Critical Implementation Rules

###1. LangGraph State Definition (MANDATORY)

```python
# CORRECT - Use TypedDict
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class TruthSeekerState(TypedDict):
 task_id: str
 messages: Annotated[list, add_messages]
 evidence_board: dict
 round_count: int

# WRONG - Never use Pydantic for State
from pydantic import BaseModel # FORBIDDEN for State!
```

###2. Motion Import Pattern

```tsx
// CORRECT
import { motion } from "motion/react"

// WRONG - Old package name
import { motion } from "framer-motion"
```

###3. Tailwind v4 Configuration

```css
/* globals.css */
@import "tailwindcss";
@import "tw-animate-css"; /* Not tailwindcss-animate */

@theme {
 --color-indigo-ai: #6366F1;
 --color-cyber-lime: #D4FF12;
}
```

###4. Supabase SSR Client

```tsx
// Use @supabase/ssr, never @supabase/auth-helpers
import { createBrowserClient } from '@supabase/ssr'
```

## Development Commands

### Frontend Setup
```bash
cd truthseeker-web

# Install dependencies
npm install

# Development server
npm run dev

# Build
npm run build

# Lint
npm run lint
```

### Backend Setup
```bash
cd truthseeker-api

# Virtual environment
python -m venv venv
source venv/bin/activate # Windows: venv\Scripts\activate


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [re-gion/TruthSeeker](https://github.com/re-gion/TruthSeeker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
