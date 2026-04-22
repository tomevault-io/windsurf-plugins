---
trigger: always_on
description: Community fork of [clusterzx/paperless-ai](https://github.com/clusterzx/paperless-ai) - an AI-powered document processing extension for Paperless-ngx. This fork focuses on integration testing, performance optimizations, and Docker improvements. Core development credit belongs to the upstream project.
---

# Copilot Instructions for Paperless-AI next

## Project Context
Community fork of [clusterzx/paperless-ai](https://github.com/clusterzx/paperless-ai) - an AI-powered document processing extension for Paperless-ngx. This fork focuses on integration testing, performance optimizations, and Docker improvements. Core development credit belongs to the upstream project.

## Architecture

### Dual Runtime System
- **Node.js (Express)**: Main API server, document processing, UI (`server.js`)
- **Python (FastAPI)**: Optional RAG service for semantic search (`main.py`)
- **Startup**: `start-services.sh` launches both with PM2 + uvicorn
- **Database**: better-sqlite3 with WAL mode (`models/document.js`)

### Service Layer Pattern
All services follow singleton pattern: `class ServiceName { ... }; module.exports = new ServiceName();`

**AI Provider Factory** (`services/aiServiceFactory.js`):
- Returns appropriate service based on `config.aiProvider` (openai|ollama|custom|azure)
- All AI services must implement: `analyzeDocument(content, doc, existingTags, correspondents)`
- Use `RestrictionPromptService.processRestrictionsInPrompt()` for placeholder replacement (`%RESTRICTED_TAGS%`, `%RESTRICTED_CORRESPONDENTS%`)

**Token Management** (`services/serviceUtils.js`):
- `calculateTokens(text, model)` - Uses tiktoken for OpenAI models, character estimation (÷4) for others
- `truncateToTokenLimit(text, maxTokens, model)` - Smart truncation with safety buffer

### Configuration System
All config loads from `data/.env` via `config/config.js`. Key patterns:
- Boolean parsing: `parseEnvBoolean(value, defaultValue)` - handles 'yes'/'no', 'true'/'false', '1'/'0'
- Feature toggles: `activateTagging`, `activateCorrespondents`, etc.
- AI restrictions: `restrictToExistingTags`, `restrictToExistingCorrespondents`

### Database Schema
5 key tables in `data/documents.db` (see `models/document.js`):
- `processed_documents` - Tracks processed docs (document_id, title)
- `history_documents` - UI history with pagination support
- `openai_metrics` - Token usage tracking
- `original_documents` - Pre-AI metadata snapshot
- `users` - Authentication (bcryptjs passwords)

**Performance Pattern**: Use prepared statements for all queries. History pagination uses SQL `LIMIT/OFFSET`, not in-memory filtering.

## Critical Workflows

### Document Processing Flow
1. `node-cron` triggers scan based on `config.scanInterval` (cron format)
2. `scanDocuments()` fetches from Paperless-ngx API
3. Retry tracking: `retryTracker` Map prevents infinite loops (max 3 attempts)
4. Content validation: Documents need ≥ `MIN_CONTENT_LENGTH` chars (default: 10)
5. **Tag filtering**: If `PROCESS_PREDEFINED_DOCUMENTS=yes`, only process docs with tags matching `TAGS` env var
6. AI service processes via factory pattern
7. Results posted back to Paperless-ngx via `paperlessService.updateDocument()`

**Key Files**: `server.js` lines 150-400, `services/paperlessService.js`

### RAG Service Integration
- Python service runs on port 8000 (configurable via `RAG_SERVICE_URL`)
- Node.js proxies requests through `services/ragService.js`
- Embeddings: sentence-transformers with ChromaDB vector store
- Hybrid search: BM25 (keyword) + semantic embeddings, weighted 30/70
- **Important**: RAG endpoints check `RAG_SERVICE_ENABLED` before proxying

### Authentication & Security
- JWT stored in cookies (`jwt` cookie name)
- API key support via `x-api-key` header
- Middleware: `isAuthenticated` checks both JWT and API key
- Protected routes use `protectApiRoute` middleware
- **Pattern**: All `/api/*` routes require authentication, including `/api-docs`

### Server-Side Pagination (PERF-001)
History table uses SQL-based pagination instead of loading all records:
```javascript
// In routes/setup.js - /api/history endpoint
const { start, length, search, order } = req.query; // DataTables params
// Use getPaginatedHistoryDocuments() with LIMIT/OFFSET
```
Tag caching with 5-minute TTL reduces Paperless-ngx API calls by ~95%.

## Development Commands

```bash
# Local development (auto-reload)
npm run test  # Uses nodemon

# Python RAG service
source venv/bin/activate
python main.py --host 127.0.0.1 --port 8000

# Production (Docker uses this)
pm2 start ecosystem.config.js

# Database inspection
sqlite3 data/documents.db "SELECT * FROM processed_documents LIMIT 5;"
```

## Code Conventions

### Language Requirement (MANDATORY)
- English only across the project.
- All source code comments, commit messages, pull requests, issue templates, documentation, UI labels, and user-facing text must be written in English.
- Do not introduce German or mixed-language content in any repository file.

### Error Handling
- Always log to both `htmlLogger` and `txtLogger` for user-facing operations
- Use try-catch in all async routes
- Return meaningful HTTP status codes (400 for validation, 500 for server errors)

### Frontend Integration
- EJS templates in `views/` (no framework)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [admonstrator/paperless-ai-next](https://github.com/admonstrator/paperless-ai-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
