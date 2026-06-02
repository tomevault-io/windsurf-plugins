---
trigger: always_on
description: The Contoso Voice Agent is a sophisticated full-stack AI-powered application that combines text chat and voice calling capabilities to provide personalized product recommendations and customer support. The system serves as a retail assistant for Contoso Outdoor Company, helping customers discover and purchase outdoor gear through natural conversations.
---

# Contoso Voice Agent - Project Context

## Project Overview

The Contoso Voice Agent is a sophisticated full-stack AI-powered application that combines text chat and voice calling capabilities to provide personalized product recommendations and customer support. The system serves as a retail assistant for Contoso Outdoor Company, helping customers discover and purchase outdoor gear through natural conversations.

## Architecture

### High-Level System Design
```
┌─────────────────┐    WebSocket/HTTP    ┌──────────────────┐
│   Next.js       │◄────────────────────►│   FastAPI        │
│   Frontend      │                      │   Backend        │
│   (Port 3000)   │                      │   (Port 8000)    │
│                 │                      │                  │
│ ┌─────────────┐ │   Real-time Audio    │                  │
│ │ Web Audio   │ │──────────────────────┤                  │
│ │ Capture &   │ │      (WebSocket)     │                  │
│ │ Processing  │ │                      │                  │
│ └─────────────┘ │                      │                  │
└─────────────────┘                      └──────────────────┘
                                                   │
                                                   │ Audio Stream
                                                   ▼
                                         ┌──────────────────┐
                                         │   Azure OpenAI   │
                                         │   GPT-4o         │
                                         │   Realtime API   │
                                         └──────────────────┘
```

### Technology Stack

**Backend (Python FastAPI)**
- **Framework**: FastAPI with WebSocket support
- **AI Integration**: Azure OpenAI GPT-4o and Realtime API
- **Prompt Management**: Prompty framework for structured prompts
- **Session Management**: Custom session handling with conversation context
- **Data Models**: Pydantic models for type safety
- **Testing**: Pytest with evaluation framework

**Frontend (Next.js TypeScript)**
- **Framework**: Next.js 14 with App Router
- **State Management**: Zustand for global state
- **Audio Processing**: Web Audio API with custom worklets
- **Real-time Communication**: WebSocket client for voice/chat
- **UI Components**: React components with TypeScript
- **Styling**: CSS modules and styled components

**AI & Data**
- **Language Model**: Azure OpenAI GPT-4o for chat and suggestions
- **Voice Processing**: Azure OpenAI Realtime API for voice conversations
- **Product Data**: JSON-based product catalog (20 outdoor products)
- **User Data**: Purchase history and preferences in JSON format
- **Prompt Engineering**: Structured prompty files for different scenarios

## Core Features

### 1. Text Chat Interface
- **Location**: `web/src/components/chat/` and `api/chat/`
- **Functionality**: Real-time text conversations with AI assistant
- **AI Integration**: Uses `chat.prompty` with product catalog integration
- **State Management**: Zustand store (`web/src/store/chat.ts`)
- **WebSocket**: Real-time message exchange via WebSocket connection

### 2. Voice Calling System
- **Location**: `web/src/components/voice/` and `api/voice/`
- **Functionality**: Real-time voice conversations with AI
- **Audio Processing**: Custom Web Audio worklets for real-time audio
- **Voice Generation**: Azure OpenAI Realtime API integration
- **State Management**: Voice-specific Zustand store (`web/src/store/voice.ts`)

### 3. Product Recommendation Engine
- **Location**: `api/suggestions/` and integrated throughout
- **Functionality**: AI-powered product suggestions based on context
- **Data Sources**: Product catalog, purchase history, conversation context
- **Prompts**: `suggestions.prompty` for generating recommendations
- **Integration**: Embedded in both chat and voice interactions

### 4. Session Management
- **Location**: `api/session.py`
- **Functionality**: Maintains conversation context and user state
- **Features**: Message history, user preferences, conversation continuity
- **Storage**: In-memory session storage with conversation persistence

## Project Structure Deep Dive

### Backend API (`/api`)

#### Core Files
- **`main.py`**: FastAPI application entry point with WebSocket endpoints
- **`models.py`**: Pydantic data models for products, messages, sessions
- **`session.py`**: Session management and conversation context
- **`requirements.txt`**: Python dependencies (FastAPI, Azure OpenAI, etc.)

#### Chat Module (`/api/chat`)
- **`chat.prompty`**: Main chat prompt template with system instructions
- **`products.json`**: Product catalog for recommendations
- **`purchases.json`**: User purchase history data
- **`schema.json`**: JSON schema definitions

#### Suggestions Module (`/api/suggestions`)
- **`suggestions.prompty`**: Product suggestion prompt template
- **`writeup.prompty`**: Product writeup generation prompt
- **Supporting JSON files**: Products, purchases, and messages data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sethjuarez/contoso-voice-agent](https://github.com/sethjuarez/contoso-voice-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
