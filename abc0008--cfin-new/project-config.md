---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CFIN (Financial Document Analysis System) is an AI-powered platform for analyzing financial documents. It combines PDF processing, conversational AI, and dynamic data visualization using Claude API as its core engine.

## Tech Stack

### Backend
- **Framework**: FastAPI (Python 3.8+)
- **Database**: SQLAlchemy with PostgreSQL/SQLite
- **AI Integration**: Anthropic Claude API (primary), LangChain/LangGraph (secondary)
- **Storage**: Local filesystem or AWS S3
- **Key Dependencies**: anthropic, langchain, fastapi, sqlalchemy, pydantic
- **Backend Start Terminal Command**: cd /Users/alexcardell/AlexCoding_Local/cfin/backend && PYTHONPATH=$PYTHONPATH:/Users/alexcardell/AlexCoding_Local/cfin/backend python -m uvicorn app.main:app --reload --port 8000

### Frontend
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **UI Components**: Tailwind CSS, shadcn/ui
- **Charts**: Recharts
- **State Management**: React hooks and context

## Development Setup

### Backend Setup
```bash
cd cfin/backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# Set environment variables
export ANTHROPIC_API_KEY="your-api-key"
export BACKEND_URL="http://localhost:8000"
export CORS_ORIGINS="http://localhost:3000"

# Initialize database
python create_db.py

# Run development server
python run.py  # or uvicorn app.main:app --reload --port 8000
```

### Frontend Setup
```bash
cd cfin/nextjs-fdas
npm install
npm run dev  # Runs on http://localhost:3000
```

## Key Development Commands

### Backend Commands
```bash
# Run backend server
python run.py
python debug_server.py  # Debug mode with verbose logging
./restart_server.sh     # Restart server (kills existing process)

# Run tests
cd backend/tests
./run_tests.sh          # Run all tests
pytest test_specific.py  # Run specific test file
pytest -k "test_name"   # Run specific test by name
pytest -v              # Verbose output

# Database operations
python create_db.py     # Initialize/reset database
python migrate_claude_fields.py  # Run migrations

# API testing scripts
./test_api.sh          # Test all endpoints
./test_document_api_only.sh  # Test document endpoints
./test_document_endpoints.sh  # Document endpoint tests
```

### Frontend Commands
```bash
# Development
npm run dev            # Start development server
npm run build         # Build for production
npm run start         # Start production server
npm run lint          # Run ESLint
npm run test          # Run Jest tests
npm test -- --watch   # Run tests in watch mode
```

## Project Structure

```
cfin/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app entry point
│   │   └── routes/              # API endpoints
│   │       ├── document.py      # Document management
│   │       ├── conversation.py  # Chat interface
│   │       └── analysis.py      # Analysis operations
│   ├── services/
│   │   ├── analysis_service.py  # Analysis orchestration
│   │   ├── conversation_service.py
│   │   └── analysis_strategies/ # Analysis implementations
│   ├── pdf_processing/
│   │   ├── claude_file_client.py # Claude Files API integration
│   │   └── financial_agent.py    # Financial analysis logic
│   ├── models/                   # Pydantic models & DB schemas
│   ├── repositories/             # Data access layer
│   └── utils/                    # Utilities and helpers
├── nextjs-fdas/
│   └── src/
│       ├── app/                  # Next.js app router pages
│       │   ├── dashboard/        # Document library
│       │   └── workspace/        # Main analysis interface
│       ├── components/           # React components
│       │   ├── chat/            # Chat interface components
│       │   ├── charts/          # Visualization components
│       │   └── document/        # Document handling
│       └── lib/api/             # API client functions
└── uploads/                     # Document storage (local mode)
```

## API Endpoints

### Document Management
- `POST /api/documents/upload` - Upload PDF with Claude processing
- `GET /api/documents/{id}` - Get document details
- `GET /api/documents/` - List all documents
- `DELETE /api/documents/{id}` - Delete document
- `GET /api/documents/{id}/citations` - Get citations for highlighting

### Conversation/Chat
- `POST /api/conversation/message` - Send message with context
- `POST /api/conversation/create` - Create new conversation
- `GET /api/conversation/{id}` - Get conversation history
- `GET /api/conversation/history` - List all conversations

### Analysis
- `POST /api/analysis/run` - Execute analysis on documents
- `GET /api/analysis/{id}` - Get analysis results
- `GET /api/analysis/types` - List available analysis types

## Claude API Configuration

The system uses different Claude models based on the operation:
- **Document Processing**: Claude 3 Haiku (fast, cost-effective)
- **Analysis & Chat**: Claude 3 Sonnet (balanced performance)
- **Complex Analysis**: Claude 3 Opus (highest capability)

Key settings in `backend/settings.py`:
- `DEFAULT_MODEL`: "claude-3-5-sonnet-20241022"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abc0008) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
