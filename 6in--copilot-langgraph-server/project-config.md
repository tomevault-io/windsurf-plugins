---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Copilot LangGraph Chat**

GitHub Copilot を LangGraph の AI プロバイダーとして使う、社内向け汎用チャット Web アプリ。
`ChatCopilot`（`BaseChatModel` のカスタム実装）を通じて Copilot の推論能力を活用しながら、LangGraph のグラフ構造により将来のエージェント化・ツール呼び出し拡張に対応できる設計を目指す。

> **利用コンテキスト:** 当初は個人用ツールとして設計されたが、現在は **社内プロジェクト向けシステム**として開発を進めている。想定ユーザー規模は **200名程度**。リクエスト量は多くないが、マルチユーザー・マルチアプリケーションの運用に耐える設計（ユーザー分離、アプリケーション管理、監査ログ）を整備中。

**Core Value:** Copilot の JSON-RPC ベース SDK を LangChain 互換プロバイダーとして動かし、アプリケーション（Chat / SuperChat）＋ユーザーという単位でスレッドを管理できるチャット UI から使えること。

### Constraints

- **Tech Stack**: Python（LangChain / LangGraph / Copilot SDK） — ドキュメントのサンプルコードが Python ベース
- **Auth**: Device Flow のみ — 非インタラクティブ環境向け PAT 方式は今回対象外
- **SDK 安定性**: Copilot SDK は Technical Preview — 外部インターフェースを薄いラッパーで隔離しておく
- **スケール感**: 200名規模・社内利用 — 高トラフィック対策より運用性（監査ログ・アプリ管理）を優先する
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Runtime
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Python | 3.12 | Runtime | 3.12 is the stable sweet spot: full support from LangGraph, FastAPI, and github-copilot-sdk (>=3.11). 3.13 is supported but less battle-tested in the ecosystem. | HIGH |
### Core AI Framework
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `langgraph` | 1.1.3 | Stateful conversation graph | The project's core orchestration layer. Provides StateGraph, MessagesState, compiled graph with checkpointer support. Production/Stable (status 5 on PyPI). Python 3.10+ required; 3.13 now officially supported. | HIGH |
| `langchain-core` | 1.2.23 | BaseChatModel base class | Required for the `ChatCopilot` custom provider. Provides `BaseChatModel`, `HumanMessage`, `AIMessage`, `ChatResult`, `ChatGeneration`. Do NOT install the full `langchain` package — `langchain-core` is the slim, stable dependency surface needed. | HIGH |
### Custom Provider
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `github-copilot-sdk` | 0.2.0 | GitHub Copilot JSON-RPC client | The SDK bundles platform-specific Copilot CLI binaries into Python wheels — no separate CLI install needed. Communicates via JSON-RPC (not HTTP/OpenAI-compatible), so `ChatOpenAI(base_url=...)` is not an option. A thin `BaseChatModel` wrapper isolates breaking changes. Technical Preview: pin to an exact version. | LOW (Technical Preview, breaking changes possible) |
### Web Backend
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `fastapi` | 0.135.2 | HTTP + WebSocket API server | FastAPI is the standard choice for async Python APIs in 2025 (78.9k GitHub stars vs Flask's 68.4k; ASGI vs WSGI). Native `async def` routes integrate naturally with LangGraph's `ainvoke`. Built-in Pydantic validation and OpenAPI docs. LangGraph + FastAPI is the most documented integration pattern in 2025. | HIGH |
| `uvicorn` | 0.42.0 | ASGI server | Standard ASGI server for FastAPI. Use `uvicorn[standard]` for production (includes `uvloop` and `httptools`). | HIGH |
### Frontend
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| React 19 + ReactDOM | 19.2 | SPA chat UI | Component-based UI with hooks. Serves the primary chat interface at `/app` via Vite dev server or built static assets. | HIGH |
| TypeScript | 5.9 | Type safety | Full type coverage for components, hooks, and API types. | HIGH |
| Vite | 8.0 | Dev server + bundler | Fast HMR in development; `bun run build` produces `frontend/dist/` served by FastAPI. Vite proxy (`/api` → `localhost:8000`) in dev. | HIGH |
| @chatscope/chat-ui-kit-react | 2.1 | Chat UI components | Ready-made chat UI (MessageList, MessageInput, TypingIndicator). Eliminates custom chat layout work. | HIGH |
| react-markdown + remark-gfm + rehype-highlight | 10.1 / 4.0 / 7.0 | Markdown rendering | Renders AI responses with GFM and syntax highlighting inside chat messages. | HIGH |
| Bun | latest | Package manager (Docker) | Used as the package manager and runtime in the `frontend` Docker service. | MEDIUM |
| Jinja2 | 3.x (FastAPI ships with it) | HTML templating | Used for the legacy Vanilla JS UI served at `/`. Zero additional dependency cost since FastAPI already pulls it in. | MEDIUM |
### Persistence / Session State
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| `langgraph-checkpoint-sqlite` | 3.0.3 | Conversation thread persistence | SQLite is the right fit for a single-user local tool: zero operational overhead (no Redis server), file-based durability across process restarts, and `AsyncSqliteSaver` integrates cleanly with async FastAPI. `MemorySaver` is in-memory only (lost on restart) — insufficient for a chat app where history should survive. Redis is explicitly ruled out in PROJECT.md ("個人ツールのため不要"). | HIGH |
### Authentication / Security
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/6in) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
