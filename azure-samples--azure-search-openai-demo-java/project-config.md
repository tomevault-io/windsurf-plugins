---
trigger: always_on
description: This is a production-ready RAG (Retrieval Augmented Generation) chat application that demonstrates best practices for creating ChatGPT-like experiences over enterprise data. The solution implements an event-driven microservices architecture using Java, React, and Azure AI services, with support for multiple deployment platforms (Azure Container Apps, Azure Kubernetes Service, and Azure App Service).
---

# Project Documentation: Azure Search OpenAI Demo Java

## Overview

This is a production-ready RAG (Retrieval Augmented Generation) chat application that demonstrates best practices for creating ChatGPT-like experiences over enterprise data. The solution implements an event-driven microservices architecture using Java, React, and Azure AI services, with support for multiple deployment platforms (Azure Container Apps, Azure Kubernetes Service, and Azure App Service).

The application serves as a fictional Contoso Electronics employee portal where users can ask questions about benefits, policies, job descriptions, and roles. It features document upload management, role-based access control with EntraID, and real-time document indexing.

---

## Technical Stack

### Backend (API Service)
- **Framework**: Spring Boot 3.3.2
- **Java Version**: 17 (Microsoft JDK)
- **AI Orchestration**: Langchain4J 1.0.1
- **Key Dependencies**:
  - `azure-ai-openai` 1.0.0-beta.16 - Azure OpenAI Service integration
  - `langchain4j-azure-open-ai` - LangChain4J Azure adapter
  - `spring-cloud-azure` 5.14.0 - Azure service integrations
  - `azure-search-documents` 11.7.2 - Azure AI Search SDK
  - Spring WebFlux - Reactive/streaming support
- **Build Tool**: Maven
- **Features**:
  - Synchronous and asynchronous streaming chat endpoints
  - RAG pattern orchestration between LLM and content retrieval
  - Response tracking with citations, search keywords, and token usage
  - CosmosDB integration for chat history

### Frontend (Web Application)
- **Framework**: React 18.3.1
- **Build Tool**: Vite 5.4.18
- **Language**: TypeScript 5.6.3
- **UI Library**: Fluent UI (React Components & Icons)
- **Key Dependencies**:
  - `@azure/msal-react` 2.2.0 - Microsoft Authentication Library
  - `@azure/msal-browser` 3.26.1 - MSAL browser support
  - `react-router-dom` 6.28.0 - Routing
  - `react-markdown` 9.0.1 - Markdown rendering
  - `react-syntax-highlighter` 15.6.1 - Code highlighting
  - `i18next` 24.2.0 - Internationalization
  - `idb` 8.0.0 - IndexedDB for browser-based chat history
- **Web Server**: Nginx (reverse proxy in production)
- **Node Version**: >= 20.0.0

### Indexer Service
- **Framework**: Spring Boot 3.3.2
- **Java Version**: 17
- **AI Components**: Langchain4J 1.0.1
- **Key Dependencies**:
  - `azure-ai-documentintelligence` 4.1.4 - Document parsing
  - `azure-sdk-bom` 1.2.33 - Azure SDK management
  - `picocli` 4.7.5 - CLI interface (for CLI module)
  - `itextpdf` 5.5.13.3 - PDF processing
- **Modules**:
  - **Core**: Shared indexing logic and utilities
  - **CLI**: Command-line interface for batch indexing
  - **Microservice**: Event-driven indexing service
  - **Functions**: Azure Functions implementation for blob processing
- **Build Tool**: Maven

### Azure Services
- **Azure OpenAI Service**: GPT-4o-mini model for chat completions
- **Azure AI Search**: Vector and hybrid search for document retrieval
- **Azure Document Intelligence**: Structured text and table extraction from PDFs
- **Azure Cosmos DB**: Chat conversation history storage
- **Azure Blob Storage**: Document storage (staging and default containers)
- **Azure Service Bus**: Message queue for async indexing triggers
- **Azure Event Grid**: Real-time blob upload notifications
- **Azure Application Insights**: Monitoring and telemetry
- **Azure Container Apps / AKS / App Service**: Hosting platforms
- **Azure Key Vault**: Secrets management
- **Azure Monitor**: Observability and diagnostics
- **EntraID (Azure Active Directory)**: Authentication and authorization

### DevOps & Infrastructure
- **IaC Tool**: Bicep (Azure native)
- **Orchestration**: Azure Developer CLI (azd)
- **Containerization**: Docker
- **Container Orchestration Options**:
  - Docker Compose (local development)
  - Azure Container Apps (serverless containers)
  - Azure Kubernetes Service (full Kubernetes)
- **CI/CD**: Azure Pipelines, GitHub Actions
- **Development Environment**: Dev Containers support

### Evaluation & Testing
- **Framework**: Python-based evaluation scripts
- **Test Types**:
  - Ground truth evaluation
  - Safety evaluation
  - Response quality metrics
- **Dependencies**: Python 3.x with custom evaluation requirements

---

## Repository Structure

### Root Level Files

| File/Folder | Description |
|-------------|-------------|
| `README.md` | Main project documentation with quick start guides |
| `CHANGELOG.md` | Version history and release notes |
| `CONTRIBUTING.md` | Contribution guidelines for developers |
| `LICENSE` / `LICENSE.md` | MIT License files |
| `SECURITY.md` | Security policy and vulnerability reporting |
| `CODEOWNERS` | GitHub code ownership definitions |
| `ps-rule.yaml` | Azure PSRule configuration for infrastructure validation |

### `/app` - Application Source Code

Main application directory containing all microservices and frontend code.

#### `/app/backend`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/azure-search-openai-demo-java](https://github.com/Azure-Samples/azure-search-openai-demo-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
