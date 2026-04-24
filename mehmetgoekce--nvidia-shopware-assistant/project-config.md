---
trigger: always_on
description: Follow this document to understand the architecture, workflow, components and details of the system
---

# NVIDIA Retail Shopping Assistant - Codebase Analysis

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture](#architecture)
3. [Core Components](#core-components)
4. [Data Flow & Workflows](#data-flow--workflows)
5. [API Endpoints](#api-endpoints)
6. [Configuration System](#configuration-system)
7. [Frontend Implementation](#frontend-implementation)
8. [Database & Storage](#database--storage)
9. [Deployment](#deployment)
10. [Key Files Reference](#key-files-reference)

## System Overview

The NVIDIA Retail Shopping Assistant is an AI-powered microservices application that provides intelligent shopping assistance through:

- **Multi-agent AI architecture** using LangGraph for orchestration
- **Natural language queries** and **image-based search**
- **Smart cart management** with persistent storage
- **Content safety** via guardrails
- **Real-time streaming responses**
- **Modern React-based UI**

### Key Technologies
- **LangGraph**: Agent orchestration framework
- **FastAPI**: Python web framework for all backend services
- **React + TypeScript**: Frontend
- **Milvus**: Vector database for similarity search
- **SQLite**: User context and cart storage
- **Docker**: Containerized deployment
- **NVIDIA NIM**: LLM and embedding models

## Architecture

### High-Level Architecture

```
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│   Frontend  │────│    Nginx     │    │  External APIs  │
│  (React)    │    │   Proxy      │    │  (NVIDIA NIM)   │
└─────────────┘    └──────────────┘    └─────────────────┘
                           │
                   ┌──────────────┐
                   │ Chain Server │ ← Main Orchestrator
                   │ (LangGraph)  │
                   └──────────────┘
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│ Catalog     │    │   Memory     │    │ Guardrails  │
│ Retriever   │    │ Retriever    │    │   (Rails)   │
└─────────────┘    └──────────────┘    └─────────────┘
       │                   │
┌─────────────┐    ┌──────────────┐
│   Milvus    │    │   SQLite     │
│  (Vector)   │    │ (Context/    │
│             │    │  Cart)       │
└─────────────┘    └──────────────┘
```

### Service Communication

All services communicate via HTTP REST APIs:
- **Port 3000**: Frontend (via Nginx)
- **Port 8009**: Chain Server (main orchestrator)
- **Port 8010**: Catalog Retriever
- **Port 8011**: Memory Retriever  
- **Port 8012**: Guardrails
- **Port 19530**: Milvus Database
- **Port 9000/9001**: Minio (object storage)
- **Port 2379**: etcd (coordination)

## Core Components

### 1. Chain Server (`/chain_server/`)

**Purpose**: Main orchestrator using LangGraph to coordinate specialized agents

**Key Files**:
- `main.py`: FastAPI application with streaming endpoints
- `graph.py`: LangGraph definition and node orchestration
- `planner.py`: Query routing agent
- `cart.py`: Shopping cart management agent
- `retriever.py`: Product search agent
- `chatter.py`: Natural language response generation
- `summarizer.py`: Response summarization

**Agent Types**:
- **PlannerAgent**: Routes queries to appropriate specialized agents
- **CartAgent**: Handles add/remove/view cart operations
- **RetrieverAgent**: Searches product catalog using text/image
- **ChatterAgent**: Generates conversational responses
- **SummaryAgent**: Summarizes final responses

### 2. Catalog Retriever (`/catalog_retriever/`)

**Purpose**: Vector-based product search using embeddings

**Key Functionality**:
- Text embedding using NVIDIA NIM models
- Image embedding using NVCLIP
- Milvus vector database queries
- Product similarity matching
- Category-based filtering

**API Endpoints**:
- `POST /query/text`: Text-based product search
- `POST /query/image`: Image + text search

### 3. Memory Retriever (`/memory_retriever/`)

**Purpose**: User context and shopping cart persistence

**Data Storage**:
- **Users Table**: `id`, `context` (conversation history)
- **CartItems Table**: `user_id`, `item`, `amount`

**API Endpoints**:
- `GET /user/{user_id}/context`: Retrieve user context
- `POST /user/{user_id}/context/add`: Add to context
- `GET /user/{user_id}/cart`: Get cart contents
- `POST /user/{user_id}/cart/add`: Add items to cart
- `POST /user/{user_id}/cart/remove`: Remove items from cart

### 4. Guardrails (`/guardrails/`)

**Purpose**: Content safety and moderation

**Functionality**:
- Input content filtering
- Output content filtering
- NVIDIA NIM-based safety checks
- Configurable safety prompts

### 5. Frontend (`/ui/`)

**Purpose**: React-based user interface

**Key Components**:
- `App.tsx`: Main application container
- `chatbox/`: Chat interface with streaming support
- `Navbar.tsx`: Navigation header
- `Apparel.tsx`: Product image display
- `hooks/useChat.ts`: Chat state management

**Features**:
- Real-time streaming chat
- Image upload and preview
- Product image rendering
- Guardrails toggle
- Message download
- Chat reset functionality

## Data Flow & Workflows

### Primary User Query Flow

```
1. User Input (Text/Image)
   ↓
2. Frontend (useChat.ts)
   ↓ HTTP POST /query/stream

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MehmetGoekce) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
