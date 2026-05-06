---
trigger: always_on
description: > **Documentation for AI agents and developers** - Technical stack, architecture patterns, and detailed repository structure for ContentFlow.
---

# AGENTS.md - Technical Architecture & Repository Structure

> **Documentation for AI agents and developers** - Technical stack, architecture patterns, and detailed repository structure for ContentFlow.

---

## 📋 Project Overview

**ContentFlow** is an enterprise-grade, cloud-native document and content processing platform built on Azure. It provides intelligent, scalable content processing pipelines powered by Azure AI services and orchestrated by Microsoft Agent Framework.

### Core Purpose
Transform unstructured content (PDFs, Word docs, Excel files, web content, etc.) into intelligent, actionable data through orchestrated AI-powered workflows.

### Architecture Pattern
- **Microservices**: Distributed services (API, Worker, Web) deployed independently
- **Event-Driven**: Queue-based task distribution with async processing
- **Cloud-Native**: Built for Azure Container Apps with managed Azure services
- **Declarative Workflows**: YAML-based pipeline definitions with Python execution

---

## 🛠️ Technical Stack

### Backend Services (Python)

#### Core Framework & API
- **Python**: 3.12+
- **FastAPI**: 0.128.0 - Modern async web framework for REST API
- **Uvicorn**: 0.40.0 - ASGI server with support for HTTP/2
- **Pydantic**: 2.12+ - Data validation and settings management
- **aiohttp/httpx**: Async HTTP clients for external service calls

#### Azure SDK Integration
- **azure-identity**: 1.25.1 - Managed identity and authentication
- **azure-storage-blob**: 12.27.1 - Blob storage for content persistence
- **azure-storage-queue**: 12.14.1 - Queue-based task distribution
- **azure-cosmos**: 4.14.3 - NoSQL database for metadata and state
- **azure-appconfiguration-provider**: 2.3.1 - Centralized configuration

#### Content Processing
- **ContentFlow Library**: Custom pipeline orchestration engine
- **40+ Executors**: Document parsing, AI analysis, embeddings, web scraping
- **Async Processing**: Built on asyncio for non-blocking operations

### Frontend (TypeScript/React)

#### Core Framework
- **React**: 18+ with TypeScript 5.8+
- **Vite**: Modern build tool with HMR and optimized production builds
- **Node.js**: 18+ runtime

#### UI & Styling
- **Shadcn/ui**: High-quality accessible component library
- **Radix UI**: Unstyled, accessible component primitives
- **Tailwind CSS**: Utility-first CSS framework
- **PostCSS**: CSS processing and optimization

#### State Management & Data
- **TanStack Query (React Query)**: Server state management and caching
- **React Hook Form**: Form state management with validation
- **Zod**: TypeScript-first schema validation

#### Specialized Libraries
- **ReactFlow**: Visual pipeline builder and graph visualization
- **Monaco Editor**: YAML/code editor with syntax highlighting
- **js-yaml**: YAML parsing and serialization
- **Lucide React**: Icon library

### Infrastructure & DevOps

#### Cloud Platform
- **Azure Container Apps**: Serverless container hosting
- **Azure Cosmos DB**: Globally distributed NoSQL database
- **Azure Blob Storage**: Scalable object storage
- **Azure Storage Queue**: Message queuing service
- **Azure AI Services**: Document Intelligence, OpenAI, Computer Vision
- **Azure App Configuration**: Centralized configuration management
- **Azure Application Insights**: Monitoring and diagnostics

#### Infrastructure as Code
- **Bicep**: Azure infrastructure templates
- **Azure Developer CLI (azd)**: Deployment orchestration
- **Docker**: Container packaging for all services

#### CI/CD & Tools
- **Git**: Version control
- **Shell Scripts**: Deployment automation
- **Azure AI Landing Zone**: Enterprise security and compliance framework

---

## 📁 Repository Structure

### Root Level Files

```
contentflow/
├── azure.yaml                 # Azure Developer CLI configuration
├── README.md                  # Project documentation
├── LICENSE                    # MIT License
├── CODE_OF_CONDUCT.md        # Community guidelines
├── SECURITY.md               # Security policies and reporting
└── assets/                   # Static assets (images, diagrams, demos)
```

**Key Files:**
- `azure.yaml`: Defines Azure deployment configuration, service mappings, and deployment hooks
- `README.md`: Main project documentation with overview, features, and quick start
- `SECURITY.md`: Security vulnerability reporting and best practices

---

### `/contentflow-api` - REST API Service

**Purpose:** Core REST API service for pipeline management, execution orchestration, and vault operations.

**Technology:** FastAPI, Python 3.12+, Azure Cosmos DB, Azure Blob Storage

```
contentflow-api/
├── main.py                   # FastAPI application entry point
├── requirements.txt          # Python dependencies
├── Dockerfile               # Container image definition
├── README.md                # API documentation
└── app/
    ├── __init__.py
    ├── dependencies.py       # Dependency injection (DB, storage clients)
    ├── settings.py          # Configuration and environment variables
    ├── startup.py           # Application lifecycle and initialization
    ├── core/                # Core utilities and shared logic
    ├── database/            # Cosmos DB client and operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/contentflow](https://github.com/Azure/contentflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
