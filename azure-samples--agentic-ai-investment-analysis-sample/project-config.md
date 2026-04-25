---
trigger: always_on
description: | Technology | Purpose | Version |
---

# Agents.md - Agentic AI Investment Analysis System

## 📚 Technical Stack

### Backend Technologies

| Technology | Purpose | Version |
|-----------|---------|---------|
| **Python** | Core backend language | 3.11+ |
| **FastAPI** | Modern async web framework | 0.116.1 |
| **Microsoft Agent Framework** | Multi-agent workflow orchestration | Latest |
| **Uvicorn** | ASGI server | 0.35.0 |
| **Pydantic** | Data validation and settings management | 2.11.7 |
| **Azure SDK** | Azure service integrations | Latest |

### Frontend Technologies

| Technology | Purpose | Version |
|-----------|---------|---------|
| **React** | UI framework | 18.x |
| **TypeScript** | Type-safe JavaScript | 5.x |
| **Vite** | Build tool and dev server | Latest |
| **Shadcn/UI** | Component library | Latest |
| **Tailwind CSS** | Utility-first CSS framework | Latest |
| **Lucide React** | Icon library | 0.462.0 |
| **TanStack Query** | Data fetching and caching | 5.56.2 |

### Azure Cloud Services

| Service | Purpose |
|---------|---------|
| **Azure OpenAI** | LLM provider for AI agents |
| **Azure Cosmos DB** | NoSQL database for documents, opportunities, and analyses |
| **Azure Blob Storage** | Document and file storage |
| **Azure Container Apps** | Containerized application hosting |
| **Azure Container Registry** | Docker image storage |
| **Azure Application Insights** | Application monitoring and telemetry |
| **Azure Log Analytics** | Centralized logging |
| **Azure App Configuration** | Configuration management |

### Infrastructure as Code

| Technology | Purpose |
|-----------|---------|
| **Bicep** | Azure infrastructure definition |
| **Bash Scripts** | Deployment automation |
| **Docker** | Application containerization |

---

## 📁 Repository Structure

### Root Directory

```
Agentic-AI-Investment-Analysis-Sample/
├── CODE_OF_CONDUCT.md      # Community guidelines
├── LICENSE                  # Project license
├── README.md               # Project overview and setup instructions
├── SUPPORT.md              # Support and contribution guidelines
├── _assets/                # Documentation and sample assets
├── api-app/                # Backend API application
├── infra/                  # Infrastructure deployment scripts
└── web-app/                # Frontend web application
```

---

### `_assets/` - Documentation Assets

Contains documentation assets, architecture diagrams, and sample analysis hypotheses.

```
_assets/
├── analysis-hypothesis-*.txt           # Sample analysis outputs for different companies
├── ARCHITECTURE_DIAGRAM.md            # Detailed system architecture documentation
├── README-sample-hypotheses.md        # Guide to sample hypothesis files
└── agentframework.svg                 # Microsoft Agent Framework logo
```

**Purpose**: Houses all non-code assets including sample data, architectural documentation, and visual aids for understanding the system.

---

### `api-app/` - Backend API Application

The FastAPI-based backend that orchestrates AI agents and manages data operations.

```
api-app/
├── Dockerfile                         # Container definition for API
├── main.py                           # FastAPI application entry point
├── requirements.txt                  # Python dependencies
├── run_investment_workflow.py        # Standalone workflow execution script
├── run_what_if_chat.py              # Standalone what-if chat script
├── app/                             # Main application package
└── infra/                           # Backend-specific infrastructure
```

#### `api-app/app/` - Application Core

**`app/core/`** - Core Configuration and Security
```
core/
├── __init__.py
├── auth.py           # Authentication and authorization logic
└── config.py         # Application configuration and settings
```

**`app/database/`** - Data Access Layer
```
database/
├── cosmos.py                    # Azure Cosmos DB client setup
└── repositories/                # Data access repositories
```
*Purpose*: Manages database connections and implements repository pattern for data access.

**`app/models/`** - Data Models
```
models/
├── __init__.py
├── _analysis_workflow_event.py   # Workflow event model
├── _analysis.py                  # Investment analysis model
├── _base.py                      # Base model classes
├── _document.py                  # Document model
├── _opportunity.py               # Investment opportunity model
├── _stream_event_message.py      # SSE stream event model
├── _user.py                      # User model
└── _what_if_message.py          # What-if chat message model
```
*Purpose*: Pydantic models for type-safe data validation and serialization/deserialization.

**`app/routers/`** - API Endpoints
```
routers/
├── analysis.py          # Analysis workflow endpoints (/analysis/*)
├── chat.py             # What-if chat endpoints (/chat/*)
├── oldchat_api.py      # Legacy chat API
└── opportunity.py      # Opportunity CRUD endpoints (/opportunities/*)
```
*Purpose*: FastAPI routers defining REST API endpoints. Handles request validation, response formatting, and error handling.

**Key Endpoints:**
- `POST /opportunities/{opportunity_id}/analyses/{analysis_id}/start/{client_id}` - Start analysis workflow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/Agentic-AI-Investment-Analysis-Sample](https://github.com/Azure-Samples/Agentic-AI-Investment-Analysis-Sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
