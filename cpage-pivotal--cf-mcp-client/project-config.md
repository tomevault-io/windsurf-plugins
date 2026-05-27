---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Build Commands

### Full Application Build
```bash
mvn clean package
```
This builds both the Spring Boot backend and Angular frontend. The frontend-maven-plugin automatically installs Node.js, runs `npm ci`, builds the Angular app, and copies the built assets to `target/classes/static`.

### Backend Only
```bash
mvn clean compile
```

### Frontend Development
```bash
cd src/main/frontend
npm start
```
Runs Angular dev server on localhost:4200 with hot reload.

### Frontend Build Only
```bash
cd src/main/frontend
npm run build
```

### Running Tests
- **Backend**: `mvn test` (note: no test files currently exist)
- **Frontend**: `cd src/main/frontend && npm test`
- **Frontend Watch Mode**: `cd src/main/frontend && npm test -- --watch`

### Running the Application Locally
```bash
mvn spring-boot:run
```
Starts the Spring Boot application on port 8080. Frontend must be built first or served separately.

## Architecture Overview

### Technology Stack
- **Backend**: Spring Boot 3.5.5, Spring AI 1.1.0, Java 21
- **Frontend**: Angular 20, Angular Material, TypeScript
- **Database**: PostgreSQL with pgvector extension for vector storage
- **AI Integration**: Model Context Protocol (MCP) clients, Agent2Agent (A2A) protocol, OpenAI models

### Backend Architecture
The application is structured around several key service layers:

#### Core Services
- **ChatService** (`src/main/java/org/tanzu/mcpclient/chat/ChatService.java`): Central orchestrator for chat interactions, manages MCP client connections, tool callbacks, and streaming responses
- **ModelDiscoveryService** (`src/main/java/org/tanzu/mcpclient/model/ModelDiscoveryService.java`): Service discovery for AI models from multiple sources including Cloud Foundry GenAI services
- **DocumentService** (`src/main/java/org/tanzu/mcpclient/document/DocumentService.java`): Handles PDF document upload, processing, and vector storage integration
- **MetricsService** (`src/main/java/org/tanzu/mcpclient/metrics/MetricsService.java`): Provides platform metrics including connected models and agents
- **McpServerService** (`src/main/java/org/tanzu/mcpclient/mcp/McpServerService.java`): Manages individual MCP server connections with protocol support

#### MCP Integration
The application dynamically connects to Model Context Protocol servers through:
- **Dual Protocol Support**: Both SSE (Server-Sent Events) and Streamable HTTP protocols
- **HTTP SSE transport**: With SSL context configuration for legacy MCP servers (tag: `mcpSseURL`)
- **Streamable HTTP**: Modern protocol with improved performance and reliability (tag: `mcpStreamableURL`)
- Automatic tool discovery and registration from MCP servers
- Session-based conversation management with tool callback providers
- **Automatic Session Recovery**: Detects and recovers from MCP server restarts by automatically reconnecting with a fresh session when "Session not found" errors occur
- **Graceful Degradation**: If an MCP server is unavailable, it is skipped and the chat service continues with available servers

#### A2A Agent Integration
The application supports Agent2Agent (A2A) protocol for communication with independent AI agent systems:
- **Protocol Differences**: Unlike MCP servers (which provide tools your LLM invokes), A2A agents are independent AI systems that process messages and return complete responses
- **Agent Discovery**: Agents register via user-provided service URLs pointing to Agent Card (JSON descriptor at `/.well-known/agent.json`)
- **Service Binding**: Use Cloud Foundry user-provided services with tag `a2a`
- **UI Integration**: Agents panel (🤖) displays connected agents with health status, capabilities, and message interface
- **Capabilities**: Supports streaming, push notifications, and state history depending on agent implementation
- **Visual Attribution**: Agent responses displayed with distinct styling and clear agent identification

#### Vector Storage
Uses PostgreSQL with pgvector extension for:
- Document embeddings and semantic search
- Conversation memory persistence across sessions
- RAG (Retrieval Augmented Generation) capabilities

### Frontend Architecture
Angular 20 standalone components architecture:

#### Key Components
- **AppComponent**: Root component managing metrics polling and document selection state
- **ChatboxComponent**: Main chat interface with SSE streaming support
- **Chat/Memory/Document/AgentsPanelComponent**: Sidebar panels for different platform aspects
- **SidenavService**: Manages exclusive sidebar navigation state

#### Communication Patterns
- **SSE Streaming**: Real-time chat responses via Server-Sent Events
- **Metrics Polling**: 5-second interval updates of platform status
- **Document Selection**: Parent-child component communication for document context

### Cloud Foundry Integration
The application is designed for Cloud Foundry deployment with service binding support:
- **GenAI Services**: Automatic discovery of chat and embedding models
- **Vector Database**: PostgreSQL service binding for vector storage
- **MCP Agents**: User-provided service URLs for external tool integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cpage-pivotal/cf-mcp-client](https://github.com/cpage-pivotal/cf-mcp-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
