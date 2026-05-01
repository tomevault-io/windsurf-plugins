---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**nokode** is an experimental web server that tests a simple question: what if you skip code generation entirely and let an LLM handle application logic directly?

This is an open source weekend experiment exploring what's possible with today's AI technology. No routes, no controllers, no business logic—just an HTTP server that hands every request to an LLM with three tools.

**Repository**: https://github.com/samrolken/nokode

## What This Tests

Everyone's focused on AI that *writes* code (Cursor, Copilot, etc.). This project tests something different: can an LLM *replace* application code by reasoning through requests and executing via tools?

Hypothesis: If the goal is fulfilling user intent, why generate code at all?

Current result: It works, but it's catastrophically slow (30-60s per request), absurdly expensive (100-1000x traditional compute), and has no UI consistency. The capability exists; performance is the limiting factor.

## Architecture

```
nokode/
├── src/
│   ├── server.js              # Express server
│   ├── config/index.js        # Model configuration
│   ├── middleware/
│   │   └── llm-handler.js     # Core: sends ALL requests to LLM
│   ├── tools/
│   │   ├── database.js        # SQL execution on SQLite
│   │   ├── webResponse.js     # HTTP response generation
│   │   └── updateMemory.js    # Feedback persistence
│   └── utils/
├── prompt.md                  # Defines what app to build
├── memory.md                  # User feedback and preferences
├── database.db                # SQLite database (AI-designed schema)
└── .env                       # API keys and model selection
```

## How It Works

Every HTTP request triggers this flow:

1. **Load Context**: Read `prompt.md` (app description) + `memory.md` (user preferences)
2. **LLM Reasoning**: AI decides what to do with the request
3. **Tool Execution**: AI calls tools as needed:
   - `database` - Execute SQL (AI designs schema, writes queries)
   - `webResponse` - Return HTML/JavaScript/JSON (AI generates content)
   - `updateMemory` - Save user feedback (natural language UI changes)
4. **Response**: Generated content sent to browser

The AI makes ALL decisions: database schema, HTML structure, API design, validation, error handling. Everything is emergent from the three tools and the prompt.

### The Feedback Loop

Every page includes a feedback widget. Users type "make buttons bigger" or "use dark theme" and the AI appends to `memory.md`, implementing changes on the next request. The app evolves based on natural language feedback.

## Commands

```bash
npm install           # Install dependencies
npm start             # Start server on :3001
```

## Environment Setup

Create `.env` file:

```env
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-3-haiku-20240307
```

**Model Options:**
- `claude-3-haiku-20240307` - Recommended (fastest, cheapest)
- `claude-3-7-sonnet-20250219` - More capable, slower
- `gpt-4o-mini` - OpenAI alternative

**Don't use reasoning models** (like gpt-5-nano) unless you want 60+ second response times.

## Customizing the Application

**The entire interface is `prompt.md`.**

Edit `prompt.md` to change:
- What kind of app it builds (contact manager, todo list, blog, etc.)
- Features and capabilities
- UI preferences and frameworks
- Behavior and constraints

Out of the box it builds a contact manager, but you can make it build anything.

## Performance Characteristics

**Reality Check:**
- **Speed**: 30-60 seconds per request (vs 10-100ms traditional)
- **Cost**: $0.01-0.05 per request with reasoning models, $0.001-0.005 with fast models (vs ~$0.00001 traditional)
- **Consistency**: Zero. UI drifts between requests.
- **Reliability**: Hallucinated SQL = 500 error. No type safety.

**Time Breakdown:**
- 75-85% LLM reasoning
- 5-10% content generation
- <1% database operations
- 10-20% wasted tool calls

**But It Works:**
- Forms submit correctly
- Data persists across restarts
- APIs return valid JSON
- User feedback gets implemented
- AI invents sensible schemas, injection-safe SQL, REST-ish patterns, responsive layouts

## Development Notes

### This Is an Experiment

- Not production software. Not even close.
- Every request costs real API tokens. Budget accordingly.
- The AI has no memory of previous UI decisions. Colors drift, layouts change.
- Response times make it nearly unusable for real interaction.

### What Works

- CRUD operations (create, read, update, delete)
- Database schema design (emergent, sensible)
- Form validation (client and server)
- API responses (JSON, HTML, redirects)
- Natural language UI customization
- Error handling for edge cases

### What Doesn't Work

- Speed (300-6000x slower than traditional)
- Cost (100-5000x more expensive)
- UI consistency (no inter-request memory)
- Reliability (hallucinations cause runtime errors)

### Prompt Engineering Findings

**What doesn't help:**
- Instructions like "THINK QUICKLY" make it slower (model meta-reasons)
- Explicit tool call optimization gets ignored

**What helps:**
- Model selection (Haiku vs reasoning models = 2-3x faster)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samrolken/nokode](https://github.com/samrolken/nokode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
