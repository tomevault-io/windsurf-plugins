---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Spring Boot-based WeChat RAG (Retrieval-Augmented Generation) agent system built with Java 21 and LangChain4j. The system consists of three main modules:

- **agent-datasync**: Data synchronization service for processing WeChat chat logs into vector embeddings
- **agent-core**: Core RAG functionality with embedding model management
- **agent-web**: Main web application that orchestrates the entire system

## Build Commands

```bash
# Build the entire project from root
mvn clean compile

# Build specific module
mvn clean compile -pl agent-datasync
mvn clean compile -pl agent-core
mvn clean compile -pl agent-web

# Run the main web application (includes all modules)
mvn spring-boot:run -pl agent-web

# Run the data sync service standalone
mvn spring-boot:run -pl agent-datasync

# Package the application
mvn clean package

# Skip tests during build
mvn clean package -DskipTests

# Run tests
mvn test

# Run from specific module directory
cd agent-web && mvn spring-boot:run
cd agent-datasync && mvn spring-boot:run
```

## Docker Commands

```bash
# Build Docker image
docker build -t wechat-rag-agent .

# Run with Docker Compose (production setup)
docker-compose up -d

# Build and run Docker images locally
cd docker && chmod +x build-docker-images.sh && ./build-docker-images.sh

# View service status
docker-compose ps

# View logs
docker-compose logs -f wechat-rag-agent

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down --volumes
```

## Multi-Module Architecture

### Module Structure
- **Root POM**: Parent aggregator with dependency management for LangChain4j 1.1.0
- **agent-web**: Main Spring Boot application (port 8080) that orchestrates the entire system
- **agent-datasync**: Standalone data ingestion service for WeChat chatlog vectorization
- **agent-core**: Shared library for embedding models and core RAG functionality

### Key Components

**agent-web module:**
- `WechatRagAgentApplication` - Main Spring Boot application entry point
- Orchestrates both data sync and RAG query functionality
- Depends on both agent-core and agent-datasync modules

**agent-datasync module:**
- `ChatlogApi` - Reactive WebClient for WeChat chatlog APIs with pagination and date filtering
- `VectorStoreFactory` - Factory pattern for Chroma/Elasticsearch vector stores using Java 21 switch expressions
- `ChatlogVectorService` - Reactive vectorization service with parallel processing and batch optimization
- `VectorStoreConfig` - Configuration properties with enum-based provider validation
- `IncrementalSyncService` - Redis-based incremental synchronization service
- `RedisSyncStateService` - Redis state management for sync checkpoints and deduplication
- `ProgressService` - Task progress tracking and monitoring

**agent-core module:**
- `EmbeddingModelFactory` - Factory for multiple embedding providers (HuggingFace, SiliconFlow)
- `SiliconflowEmbeddingModel` - Custom embedding model implementation with WebClient integration
- `EmbeddingConfig` - Configuration properties for embedding providers
- `RerankModelFactory` - Factory for reranking model providers (SiliconFlow, Local)
- `SiliconflowRerankModel` - Custom rerank model implementation
- `Assistant` - Core RAG assistant with content injection and retrieval
- `QueryParser` - Natural language query processing and intent recognition
- `TimeParser` - Temporal expression parsing for time-based queries
- `EnhancedContentRetriever` - Advanced content retrieval with reranking

## Configuration Patterns

### Main Application Configuration (agent-web/src/main/resources/application.yml)
```yaml
server:
  port: 8080

spring:
  application:
    name: wechat-rag-agent
  data:
    redis:
      host: localhost
      port: 6379

wechat:
  chatlog:
    base-url: http://127.0.0.1:5030

rag:
  parse-model:
    api-key: your-openrouter-api-key
    model: google/gemini-2.5-flash
  embedding:
    provider: siliconflow
    model: BAAI/bge-m3
    api-key: your-siliconflow-api-key
    base-url: https://api.siliconflow.cn/v1
  rerank:
    provider: siliconflow
    model: BAAI/bge-reranker-v2-m3
    api-key: your-siliconflow-api-key
    base-url: https://api.siliconflow.cn/v1
  vector-store:
    provider: elasticsearch
    url: http://localhost:9200
    collection-name: wechat_chatlog
```

### Vector Store Configuration (`rag.vector-store`)
```properties
rag.vector-store.provider=chroma|elasticsearch
rag.vector-store.url=http://localhost:8000
rag.vector-store.collection-name=wechat-logs
```

### Embedding Model Configuration (`rag.embedding`)
```properties
rag.embedding.provider=huggingface|siliconflow|local
rag.embedding.base-url=https://api.endpoint.url
rag.embedding.api-key=your-key
rag.embedding.model=model-name
```

### Rerank Model Configuration (`rag.rerank`)
```properties
rag.rerank.provider=siliconflow|local
rag.rerank.base-url=https://api.siliconflow.cn/v1
rag.rerank.api-key=your-key
rag.rerank.model=BAAI/bge-reranker-v2-m3
```

### Redis Configuration (Spring Data Redis)
```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      timeout: 2000ms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WechatRagAgent/agent](https://github.com/WechatRagAgent/agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
