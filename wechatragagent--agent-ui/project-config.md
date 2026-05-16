---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# 沟通规范

- 所有内容必须使用 **中文** 交流（包括代码注释），但是文案与错误提示要使用英文。
- 遇到不清楚的内容应立即向用户提问。
- 表达清晰、简洁、技术准确。
- 在代码中应添加必要的注释解释关键逻辑。

## Commands

### Development
- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

### Docker
- `docker build -t agent-ui .` - Build Docker image
- `docker run -p 3000:3000 agent-ui` - Run container (standalone)
- `docker-compose up` - Start frontend only (requires backend running)
- `docker-compose up --build` - Rebuild and start frontend
- `docker-compose up` - Start frontend container in wechat-rag network

### TypeScript
- Type checking is configured via `tsconfig.json`
- Uses strict TypeScript with Next.js plugin
- Path alias `@/*` maps to root directory for imports
- No test framework currently configured

### Environment Variables
- `WECHAT_API_URL` - WeChat API endpoint (default: http://127.0.0.1:5030)
- `BACKEND_API_URL` - Backend service endpoint (default: http://127.0.0.1:8080)
- `BACKEND_URL` - Alternative backend URL used in API routes (default: http://127.0.0.1:8080)
- `OPENROUTER_API_KEY` - OpenRouter API key for LLM services

**Note**: Runtime environment variables are provided to client-side components via `/api/config` endpoint. This allows containers to read Docker environment variables at runtime rather than build time.

## Architecture

This is a WeChat RAG (Retrieval-Augmented Generation) chatbot frontend built with Next.js 15, React 19, and TypeScript. The application provides a UI for interacting with WeChat chat history through an AI-powered chat interface.

### Core Architecture Patterns

**State Management**: Centralized context-based architecture using `AppProvider` in `contexts/app-context.tsx`. All major application state (settings, UI state, chat, vectorization, WeChat) is managed through this provider.

**Hook-Based Services**: Business logic is encapsulated in custom hooks:
- `use-chat.ts` - Chat functionality with streaming responses via SSE
- `use-vectorization.ts` - Background vectorization progress tracking
- `use-wechat.ts` - WeChat API integration
- `use-settings.ts` - Persistent settings management via localStorage
- `use-mobile.ts` - Mobile device detection

**Component Structure**:
- `components/setup/` - Setup wizard components
- `components/chat/` - Chat interface and message handling  
- `components/sync/` - Vectorization/sync progress monitoring
- `components/help/` - FAQ and help components
- `components/ui/` - Reusable UI components (shadcn/ui based)

**API Integration**: RESTful API layer in `lib/api/` with separate modules for:
- `chat.ts` - OpenRouter API for LLM interactions
- `vectorization.ts` - Background processing with SSE progress updates
- `wechat.ts` - WeChat data retrieval and chatroom management
- `base.ts` - Shared HTTP client utilities

### Key Data Flow

1. **Setup Flow**: Users configure API endpoints and credentials through setup wizard
2. **Chat Flow**: Messages are sent to backend, which retrieves relevant WeChat context and forwards to LLM
3. **Vectorization Flow**: Background processing of WeChat data with real-time progress updates via SSE
4. **Persistence**: Settings stored in localStorage, conversations cached locally

### Technology Stack

- **UI Framework**: Next.js 15 with App Router, React 19
- **Styling**: Tailwind CSS v4 with shadcn/ui components and tw-animate-css
- **State**: React Context + custom hooks
- **Forms**: React Hook Form with Zod validation
- **HTTP**: Fetch API with streaming support
- **Icons**: Lucide React
- **Markdown**: react-markdown with remark-gfm for GFM support
- **Notifications**: Sonner for toast notifications
- **Themes**: next-themes for dark/light mode support

### File Organization

- `/app` - Next.js app router pages, layouts, and API routes
- `/components` - React components organized by feature (chat, setup, sync, help, ui)
- `/contexts` - React context providers (centralized app state)
- `/hooks` - Custom React hooks for business logic
- `/lib` - Utilities, types, API clients, and shared constants
- `/doc` - Chinese documentation for the project

The application supports Chinese language and is specifically designed for WeChat chat analysis and interaction.

## Development Patterns

### Code Style
- Use TypeScript with strict mode enabled
- Follow React 19 patterns with hooks and functional components
- Prefer functional programming patterns over class-based components
- Use Zod for runtime type validation in forms
- Event handling with Server-Sent Events (SSE) for real-time updates

### State Management Pattern
- All major state flows through the `AppProvider` context
- Hooks encapsulate business logic and API interactions
- Settings persist to localStorage automatically
- Conversations are cached locally with auto-save functionality

### API Communication
- RESTful endpoints for standard operations
- Server-Sent Events (SSE) for streaming chat responses and progress updates
- Error handling is centralized through context providers
- Connection testing utilities available in `base.ts`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WechatRagAgent/agent-ui](https://github.com/WechatRagAgent/agent-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
