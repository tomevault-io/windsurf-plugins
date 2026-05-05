---
trigger: always_on
description: This is an advanced AI-powered deep research system built with Mastra framework, designed for intelligent, autonomous research and reporting. The system leverages multiple AI agents, sophisticated workflows, and cutting-edge RAG (Retrieval-Augmented Generation) capabilities to conduct comprehensive research with human-in-the-loop approval.
---

# Deep Research Assistant - Project Context

## Project Overview
This is an advanced AI-powered deep research system built with Mastra framework, designed for intelligent, autonomous research and reporting. The system leverages multiple AI agents, sophisticated workflows, and cutting-edge RAG (Retrieval-Augmented Generation) capabilities to conduct comprehensive research with human-in-the-loop approval.

## Core Technologies & Architecture

### Primary Framework
- **Mastra**: AI orchestration framework for agents, workflows, and tools
- **TypeScript**: Primary programming language with strict type safety
- **Node.js 20.9+**: Runtime environment with modern ES modules

### AI & ML Stack
- **Google AI Gemini 2.5**: Advanced multimodal AI with search grounding and caching
- **AI SDK**: Unified interface for multiple AI providers
- **Exa AI**: Primary web search and content discovery API
- **Vector Embeddings**: Semantic search and similarity matching
- **Graph RAG**: Knowledge graph-based reasoning and retrieval

### Data & Storage
- **LibSQL**: High-performance SQLite-compatible database (currently in-memory configuration)
- **Vector Indexes**: Multiple specialized indexes for different content types
- **Memory Processors**: 11 specialized processors for context optimization

### Observability & Monitoring
- **OpenTelemetry**: Comprehensive tracing with child spans
- **Performance Metrics**: Processing time, resource usage, API costs
- **Error Tracking**: Structured error logging and failure analysis

## Key Components

### 🤖 AI Agents (6 Specialized Agents)
1. **Research Agent**: Multi-phase research with web search and evaluation
2. **Report Agent**: Comprehensive report generation from research data
3. **Evaluation Agent**: Content relevance and quality assessment
4. **Learning Extraction Agent**: Key insights and follow-up question generation
5. **Web Summarization Agent**: Content condensation and synthesis
6. **RAG Agent**: Vector search and retrieval-augmented generation

### 🔄 Workflows (3 Core Workflows)
1. **Comprehensive Research Workflow**: End-to-end research with human approval
2. **Generate Report Workflow**: Automated report creation
3. **Research Workflow**: Basic research orchestration

### 🛠️ Specialized Tools (8+ Tools)
- **Web Search Tool**: Intelligent web scraping and content extraction
- **Vector Query Tool**: Semantic search over embedded content
- **Chunker Tool**: Document segmentation for RAG
- **Rerank Tool**: Result relevance optimization
- **Evaluation Tool**: Content quality assessment
- **Learning Extraction Tool**: Insight mining from research data

### 🧠 Memory Processors (11 Specialized Processors)
- **TokenLimiterProcessor**: Prevents context overflow
- **PersonalizationProcessor**: Boosts user-relevant content
- **ErrorCorrectionProcessor**: Deduplicates content via checksum
- **HierarchicalMemoryProcessor**: Filters episodic vs semantic content
- **CitationExtractorProcessor**: Prioritizes cited content
- **MultiPerspectiveProcessor**: Multi-viewpoint evaluation
- **TemporalReasoningProcessor**: Time-based relationship analysis
- **UncertaintyQuantificationProcessor**: Confidence scoring
- **KnowledgeGraphProcessor**: Graph construction and reasoning
- **BayesianBeliefProcessor**: Probabilistic belief updating
- **CircuitBreakerProcessor**: Fault tolerance and recovery

## Development Guidelines

### Code Quality Standards
- **TypeScript Strict Mode**: All code must be fully typed with no `any` types
- **ESLint**: Code quality and consistency enforcement
- **Zod Validation**: Runtime type validation for all data structures
- **Error Handling**: Comprehensive error handling with proper logging
- **Documentation**: JSDoc comments for all public APIs and complex functions

### Architecture Patterns
- **Agent-Based Architecture**: Modular, specialized AI agents
- **Workflow Orchestration**: Declarative workflow definitions
- **Tool Composition**: Reusable, composable tools
- **Memory Management**: Hierarchical memory with specialized processors
- **Event-Driven**: Async/await patterns with proper error propagation

### Performance Considerations
- **Token Optimization**: Efficient context management and compression
- **Caching Strategy**: Multi-level caching (explicit, implicit, search)
- **Batch Processing**: Optimized for bulk operations
- **Resource Monitoring**: Comprehensive performance tracking
- **Cost Optimization**: API usage monitoring and optimization

## Project Structure

```
deep-research/
├── src/mastra/           # Core framework code
│   ├── index.ts         # Main entry point
│   ├── agents/          # AI agent implementations
│   ├── workflows/       # Workflow definitions
│   ├── tools/           # Specialized tools
│   ├── networks/        # Multi-agent networks
│   ├── config/          # Configuration modules
│   └── mcp/            # MCP server implementation
├── docs/                # Documentation
├── templates/           # Code templates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ssdeanx/deep-research](https://github.com/ssdeanx/deep-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
