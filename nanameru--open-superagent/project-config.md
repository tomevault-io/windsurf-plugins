---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Development
**重要**: 以下の2つのサーバーを別々のターミナルで起動する必要があります：

#### Terminal 1: Mastraサーバー（MCP統合に必須）
```bash
# Mastra development server (ポート4111で起動)
npm run dev:mastra
# または
mastra dev
```

#### Terminal 2: Next.jsアプリケーションサーバー
```bash
# Start the development server (uses Turbopack)
npm run dev
```

#### その他のコマンド
```bash
# Build Mastra agents
mastra build
```

### Build & Production
```bash
# Build the application
npm build

# Start production server
npm start

# Run linting
npm run lint
```

### Common Port Issues
If you encounter `EADDRINUSE` errors:
```bash
# Find process using port (e.g., 4114)
lsof -i :4114 | grep LISTEN | cat

# Kill the process (replace PID)
kill -9 <PID>

# Restart Mastra
mastra dev
```

## High-Level Architecture

### Technology Stack
- **Frontend**: Next.js 15 (App Router), React 19, TailwindCSS, shadcn/ui
- **Backend**: Mastra agent framework
- **AI Services**: OpenAI GPT-4.1, Anthropic Claude, Google Gemini, X.AI Grok
- **Database**: LibSQL (SQLite) for Mastra memory storage
- **Deployment**: Optimized for Vercel

### Core Agent System
The application uses Mastra's agent framework with three main agents:
1. **slideCreatorAgent (Open-SuperAgent)**: The primary agent with access to all tools
2. **imageCreatorAgent**: Specialized for image generation tasks
3. **weatherAgent**: Basic weather information agent

### Tool Ecosystem
Tools are modular and located in `src/mastra/tools/`:
- **Presentation Tools**: `htmlSlideTool` (12 layouts, 11 diagram types), `presentationPreviewTool`
- **Media Generation**: Gemini/Imagen4 for images, Gemini for video, MiniMax for TTS
- **Browser Automation**: Complete Browserbase integration with stealth mode and CAPTCHA solving
- **Search**: Brave Search API, Grok X search
- **Code Generation**: V0 code generation tool
- **MCP Tools**: Dynamic integration with Model Context Protocol servers

### API Routes Structure
All API endpoints are in `app/api/`:
- `/chat`: Main chat endpoint with streaming support
- `/slide-creator/chat`: Specialized presentation chat interface
- `/export-pptx*`: Four different PPTX export methods
- `/media/*`: Image, video, and music generation endpoints

### Mastra Integration Points
1. **Configuration**: `mastra.config.ts` and `src/mastra/index.ts`
2. **Memory**: Conversation history stored in `.mastra/memory.db`
3. **Telemetry**: Enabled for monitoring agent executions
4. **Server**: Runs on port 4111 with 120-second timeout

### Environment Variables Required
The application requires multiple API keys:
```bash
# Core AI Services
OPENAI_API_KEY
ANTHROPIC_API_KEY
GOOGLE_GENERATIVE_AI_API_KEY / GEMINI_API_KEY

# Specialized Services
BROWSERBASE_API_KEY
BROWSERBASE_PROJECT_ID
XAI_API_KEY
BRAVE_API_KEY
V0_API_KEY
FAL_KEY
NUTRIENT_API_KEY
MINIMAX_API_KEY
MINIMAX_GROUP_ID

# MCP Services (Optional)
GITHUB_TOKEN        # For GitHub MCP server
TAVILY_API_KEY      # For web search MCP server
```

### PPTX Export Methods
1. **Basic**: Image-based export using html2canvas
2. **Advanced**: HTML parsing with direct PPTX generation
3. **Hybrid**: Combines both approaches
4. **Nutrient API**: Professional-grade conversion (recommended)

### License Structure
- **Project Code**: MIT License with Commercial Use Restrictions
- **Mastra Framework**: Elastic License 2.0 (ELv2)
- Commercial use restricted to AI Freak Summit/AIで遊ぼう community members

## Development Workflow

### Quick Start
1. **Terminal 1**: `npm run dev:mastra` (Mastraサーバー起動)
2. **Terminal 2**: `npm run dev` (Next.jsアプリ起動)
3. **Chrome MCP** (オプション): Chrome拡張機能をインストールして「Connect」をクリック

### Adding New Tools
1. Create tool file in `src/mastra/tools/`
2. Export from `src/mastra/tools/index.ts`
3. Register in `src/mastra/index.ts`
4. Add to relevant agents in `src/mastra/agents/`

### MCP (Model Context Protocol) Integration

The application now supports MCP servers for extended capabilities:

#### Available MCP Servers
1. **Filesystem Server**: File operations in the current directory
2. **GitHub Server**: Repository operations (requires GITHUB_TOKEN)
3. **Sequential Thinking**: Complex reasoning tasks
4. **Memory Server**: Persistent storage capabilities
5. **Web Search**: Tavily search integration (requires TAVILY_API_KEY)
6. **Chrome MCP Server**: Control your Chrome browser with AI - 20+ tools for automation, screenshots, network monitoring, and more

#### Configuration
MCP servers are configured in `src/mastra/config/mcp.config.ts`. The system automatically:
- Detects available API keys from environment variables
- Initializes only the servers with valid credentials
- Dynamically adds MCP tools to the slideCreatorAgent

#### Adding New MCP Servers
To add a new MCP server:
1. Add the server configuration to `mcpServerConfigs` in `mcp.config.ts`
2. Include any required environment variables
3. The tools will be automatically available to agents

Example:
```typescript
newServer: process.env.NEW_API_KEY ? {
  command: 'npx',
  args: ['-y', '@modelcontextprotocol/server-new'],
  env: {
    ...process.env,
    NEW_API_KEY: process.env.NEW_API_KEY,
  },
} : undefined,
```

#### Chrome MCP Server Setup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanameru/Open_SuperAgent](https://github.com/nanameru/Open_SuperAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
