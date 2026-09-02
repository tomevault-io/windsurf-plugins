---
trigger: always_on
description: **MamboStudio** is an enterprise-grade AI code generation platform built with **Spring Boot 3 + LangChain4j + Vue 3**. This is a **backend-driven architecture** where AI logic, workflows, and business features are implemented in Java, with the frontend serving as an adaptive interface layer.
---

                   # Mambo AI Code Generation Platform - AI Agent Instructions

**MamboStudio** is an enterprise-grade AI code generation platform built with **Spring Boot 3 + LangChain4j + Vue 3**. This is a **backend-driven architecture** where AI logic, workflows, and business features are implemented in Java, with the frontend serving as an adaptive interface layer.

## 🏗️ Architecture Overview

### Backend (Spring Boot 3 + Java 21) - **PRIMARY FOCUS**

**Core Technologies:**

- **AI Framework**: LangChain4j with multi-model support (ModelScope API for local dev, production models for deployment)
- **Workflow Engine**: LangGraph4j for stateful AI workflows with node-based processing
- **Data Persistence**: MyBatis-Flex (code generation via `MyBatisCodeGenerator.main()`)
- **Caching/Sessions**: Redis (dual-purpose: sessions + LangChain4j chat memory + Redisson rate limiting)
- **File Storage**: Aliyun OSS for generated code and assets, local filesystem at `tmp/code_output/`
- **Monitoring**: Prometheus + Grafana for AI model metrics and business KPIs

**Key Design Patterns:**

- **Factory Pattern**: `AiCodeGeneratorServiceFactory` with Caffeine caching (30min TTL, per-app isolation)
- **Facade Pattern**: `AiCodeGeneratorFacade` as unified entry point for code generation
- **Tool System**: `ToolManager` auto-registers Spring beans extending `BaseTool` for LangChain4j function calling
- **Prototype Scope**: AI model beans use prototype scope to prevent concurrency issues

### Frontend (Vue 3 + TypeScript) - **ADAPTATION LAYER**

**Core Technologies:**

- **Framework**: Vue 3 + Vite + TypeScript with Ant Design Vue (requires heavy customization)
- **API Client**: Auto-generated from OpenAPI spec via `npm run openapi2ts`
- **Real-time**: EventSource/SSE for streaming AI responses in `AppChatPage.vue`
- **State Management**: Pinia stores for user sessions and application state
- **Visual Editing**: Custom `VisualEditor` class for iframe-based interactive element selection

**Visual Editing System (Unique Feature):**

- **Cross-frame Communication**: PostMessage API bridges parent window and iframe preview
- **Dynamic Script Injection**: Runtime injection of editing scripts into generated HTML
- **Element Detection**: CSS hover/click handlers with intelligent selector generation
- **AI Integration**: Selected elements feed context to LangChain4j tools for precise modifications

## 🔧 Critical Development Patterns

### 1. LangChain4j Service Factory Pattern

Each app gets isolated AI services with independent chat memory:

```java
// Factory creates cached instances per {appId}_{codeGenType}
AiCodeGeneratorService service = aiCodeGeneratorServiceFactory
    .getAiCodeGeneratorService(appId, CodeGenTypeEnum.VUE_PROJECT);
```

**Key Implementation Details:**

- **Caffeine Cache**: 30min write expiration, 10min access expiration, max 1000 instances
- **Redis Chat Memory**: `MessageWindowChatMemory` backed by `RedisChatMemoryStore` (100 message window)
- **Chat History Loading**: `chatHistoryService.loadChatHistoryToMemory(appId, chatMemory, 50)` preloads context
- **Prototype Scope**: AI model beans **must** use `@Scope("prototype")` to avoid threading conflicts
- **Model Selection**: Different generation types use different models (reasoning/streaming/routing)

### 2. LangGraph4j Workflow Architecture

Stateful workflows with serializable context passing between nodes:

```java
// WorkflowContext serializes into MessagesState for node communication
WorkflowContext context = WorkflowContext.getContext(state);
context.setEnhancedPrompt("improved prompt...");
return WorkflowContext.saveContext(context);
```

**Workflow Capabilities:**

- **Nodes**: ImageCollector → PromptEnhancer → Router → CodeGenerator → QualityCheck → ProjectBuilder
- **Conditional Edges**: `edge_async()` for quality checks and retry logic with `routeAfterQualityCheck()`
- **Subgraphs**: Parallel image collection (4 concurrent subgraphs for content/illustrations/diagrams/logos)
- **SSE Streaming**: `Flux<String>` support for real-time workflow progress updates
- **Graph Visualization**: `workflow.getGraph(GraphRepresentation.Type.MERMAID)` for debugging

### 3. LangChain4j Tool System

Auto-registered tools for AI function calling via Spring's component scanning:

```java
@Component
public class FileWriteTool extends BaseTool {
    @Tool("写入文件到指定路径")
    public String writeFile(
        @P("文件的相对路径") String relativeFilePath,
        @P("要写入文件的内容") String content,
        @ToolMemoryId Long appId
    ) {
        // Resolves relative paths to vue_project_{appId}/ directory
        // Returns relative path to prevent exposing absolute paths to AI
    }
}
```

**Available Tools:**

- **File Operations**: `FileWriteTool`, `FileReadTool`, `FileModifyTool`, `FileDeleteTool`, `FileDirReadTool`
- **Image Generation**: `LogoGeneratorTool` (DashScope API), `UndrawIllustrationTool` (free SVG library)
- **Diagram Generation**: `MermaidDiagramTool` (converts Mermaid syntax to images via CLI)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Marisalice114/mambo-studio](https://github.com/Marisalice114/mambo-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
