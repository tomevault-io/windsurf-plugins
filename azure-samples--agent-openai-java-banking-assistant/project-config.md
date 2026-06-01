---
trigger: always_on
description: - **Java 17+**: Primary programming language for backend services
---

# Multi-Agent Banking Assistant - Technical Documentation

## Technical Stack

### Backend Technologies

#### Core Framework
- **Java 17+**: Primary programming language for backend services
- **Spring Boot 3.3.6**: Application framework for microservices architecture
- **Spring Security**: Authentication and authorization implementation
- **Maven**: Build and dependency management tool

#### AI & Agent Framework
- **Langchain4j 1.0.0-beta2**: Agent orchestration and tool invocation framework
  - Multi-agent supervisor pattern implementation
  - Automatic tool binding and execution
  - Conversation memory and context management
- **Azure OpenAI Service**: LLM provider
  - Models: GPT-4o-mini or GPT-4o
  - Chat completion API integration
- **Spring AI MCP (Model Context Protocol)**: Framework for exposing business APIs as agent tools
  - Automatic tool registration from REST APIs
  - Tool parameter extraction and validation

#### Azure Services
- **Azure Document Intelligence**: OCR and document data extraction
  - Prebuilt invoice model for payment processing
- **Azure Container Apps**: Microservices hosting platform
- **Azure Container Registry**: Docker image repository
- **Azure Key Vault**: Secrets and configuration management
- **Azure Application Insights**: Application performance monitoring and telemetry
- **Azure Log Analytics**: Centralized logging and diagnostics
- **Azure Cognitive Search**: (Infrastructure provisioned for future enhancements)

#### Infrastructure & DevOps
- **Azure Bicep**: Infrastructure as Code (IaC) templates
- **Azure Developer CLI (azd)**: Deployment automation and environment management
- **Docker & Docker Compose**: Container orchestration for local development
- **GitHub Actions**: CI/CD pipeline automation

### Frontend Technologies

#### Core Stack
- **React 18**: UI library for building component-based interfaces
- **TypeScript 5.2**: Type-safe JavaScript for development
- **Vite 6.3**: Modern build tool and development server

#### UI Components & Libraries
- **Fluent UI v8 & v9**: Microsoft's design system
  - @fluentui/react: UI components library
  - @fluentui/react-components: Next-generation components
  - @fluentui/react-icons: Icon library
- **React Router DOM 6**: Client-side routing
- **@react-spring/web**: Animation library

#### Authentication
- **Azure MSAL (Microsoft Authentication Library)**
  - @azure/msal-browser: Browser-based authentication
  - @azure/msal-react: React integration for authentication flows

#### Build & Development
- **Node.js >=14.0.0**: JavaScript runtime
- **Nginx**: Web server for production deployment
- **Prettier**: Code formatting

### Testing & Quality
- **JUnit 5**: Unit testing framework
- **Mockito**: Mocking framework for Java tests
- **AssertJ**: Fluent assertion library
- **WireMock**: HTTP service mocking for integration tests

---

## Project Repository Structure

### Root Level

#### Configuration Files
- **`azure.yaml`**: Azure Developer CLI configuration defining services, languages, and hosting targets
- **`compose.yaml`**: Docker Compose orchestration for local multi-container development
- **`README.md`**: Project overview, architecture, and getting started guide
- **`CHANGELOG.md`**: Version history and release notes
- **`CONTRIBUTING.md`**: Contribution guidelines for developers
- **`SECURITY.md`**: Security policies and vulnerability reporting
- **`LICENSE`** / **`LICENSE.md`**: MIT license terms
- **`CODEOWNERS`**: Code ownership and review assignment

### `/app` - Application Source Code

Main application directory containing all microservices and frontend code.

#### `/app/copilot` - AI Agent Backend Service
Multi-module Maven project implementing the multi-agent banking assistant.

- **`/copilot-backend`**: Spring Boot application serving as the main copilot service
  - REST API endpoints for chat interactions
  - Supervisor agent orchestration
  - Integration with Azure OpenAI and Document Intelligence
  - `/manifests`: Kubernetes deployment configurations for AKS
    - Backend deployment templates
    - Service definitions
    - Ingress rules
    - Environment configuration maps

- **`/langchain4j-agents`**: Agent implementation module
  - Domain-specific agents (Account, Transactions, Payments)
  - Langchain4j integration and configuration
  - MCP tool bindings and execution logic
  - Agent conversation flows and tool invocation

- **`/copilot-common`**: Shared utilities and domain models
  - Common DTOs and domain entities
  - Utility classes and helpers
  - Shared configuration

- **Root files**:
  - `pom.xml`: Parent POM coordinating all copilot modules
  - `Dockerfile`: Container image build instructions
  - `applicationinsights.json`: Application Insights configuration
  - `mvnw` / `mvnw.cmd`: Maven wrapper scripts

#### `/app/business-api` - Microservices Backend

##### `/app/business-api/account` - Account Service
- **Purpose**: Manages user account information, balances, and payment methods
- **Exposed as**: REST API and MCP tools
- **Features**:
  - Retrieve account details by username
  - Get credit balance information
  - Manage registered payment methods
  - List beneficiaries
- **Technology**: Spring Boot microservice with Maven build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/agent-openai-java-banking-assistant](https://github.com/Azure-Samples/agent-openai-java-banking-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
