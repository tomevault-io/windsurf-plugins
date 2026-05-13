---
trigger: always_on
description: QuizZySmart is a RAG-based (Retrieval-Augmented Generation) quiz and Q&A application focused on Vietnamese banking regulations. The system combines:
---

# AI Coding Agent Instructions - QuizZySmart

## Project Overview
QuizZySmart is a RAG-based (Retrieval-Augmented Generation) quiz and Q&A application focused on Vietnamese banking regulations. The system combines:
- **Frontend**: React + TypeScript (Vite)
- **Backend**: Node.js + Express + TypeScript
- **Database**: PostgreSQL + Prisma ORM
- **Vector DB**: Qdrant Cloud for semantic search
- **AI**: Google Gemini (multiple models with rotation)

## Architecture Principles

### 1. **Dual Server Architecture**
- **Client**: Vite dev server (port 5173) - React SPA
- **Server**: Express API (port 3000) - Backend services
- Commands: `npm run dev` (client) and `cd server && npm run dev` (server)
- **Never restart servers** - user runs them in their environment

### 2. **RAG Pipeline** (Core Feature)
The RAG system processes questions through multiple stages:

```
User Question → Query Preprocessing → Embedding → Vector Search → Reranking → Answer Generation
```

#### **Query Preprocessing** (NEW - Critical for Vietnamese Legal Documents)
Located in `server/src/services/query-preprocessor.service.ts`:
- **Purpose**: Transforms natural language questions into legal document style
- **Why**: Vietnamese legal documents use formal terminology that differs from casual questions
- **Process**:
  1. Analyzes user question using Gemini (cheaper model)
  2. Generates 2-4 simplified query variants matching legal terminology
  3. Example: "Tôi muốn vay 500 triệu" → ["Điều kiện vay vốn", "Quy định cho vay", "Yêu cầu khách hàng vay"]
  4. Creates embeddings for all variants
  5. For complex/low-confidence queries: searches with multiple variants and merges results

**Key Files**:
- `query-preprocessor.service.ts` - Query transformation logic
- `query-analyzer.service.ts` - Collection selection (which documents to search)
- `gemini-rag.service.ts` - Embedding generation and answer synthesis
- `qdrant.service.ts` - Vector search and reranking

### 3. **Model Management**
- **Model Rotation**: Automatically rotates between Gemini models (2.0-flash, 1.5-flash, 1.5-pro) to manage API limits
- **Cost Optimization**: Uses cheaper models (2.0-flash-lite) for query preprocessing/analysis
- **Configuration**: `server/src/gemini-model-rotation.ts` and `model-settings.service.ts`
- **Admin Control**: Can disable rotation via system settings

### 4. **Search Strategy**
Two modes controlled by query complexity:
- **Simple Query** (topK=12): Direct search, fast response
- **Complex Query** (topK=20): Multi-variant search with broader context
  - Triggered by keywords: "bao nhiêu", "tính tổng", "tóm tắt", "liệt kê tất cả"
  - Uses all query variants for comprehensive results

### 5. **Caching Layers**
- **Chat Cache** (`chat-cache.service.ts`): In-memory LRU cache for frequent questions
- **Query Preprocessing Cache**: Caches preprocessed query variants to reduce AI calls
- **Admin endpoints**: `/api/chat/cache/stats` and `/api/chat/cache/clear`

## Development Patterns

### File Organization
```
server/src/
├── index.ts              # Main Express server
├── routes/
│   ├── chat.routes.ts    # RAG Q&A endpoints
│   ├── documents.routes.ts # PDF upload/processing
│   └── auth.routes.ts    # Authentication
├── services/
│   ├── query-preprocessor.service.ts # NEW: Query transformation
│   ├── query-analyzer.service.ts     # Collection selection
│   ├── gemini-rag.service.ts         # Embeddings & answers
│   ├── qdrant.service.ts             # Vector search
│   ├── chat-cache.service.ts         # Response caching
│   └── pdf-processor.service.ts      # PDF to chunks
└── types/
    └── rag.types.ts      # TypeScript interfaces
```

### Key Endpoints
- `POST /api/chat/ask-stream` - Streaming RAG Q&A (SSE)
- `POST /api/chat/ask` - Non-streaming RAG Q&A
- `POST /api/chat/deep-search` - Enhanced search with more chunks
- `POST /api/search/image` - Camera-based question search
- `POST /api/documents/upload` - Upload PDF regulations

### Database Models (Prisma)
- **User** - Authentication, role-based access, quotas
- **KnowledgeBase** - Question banks
- **Question** - Quiz questions with embeddings
- **Document** - Uploaded PDFs (metadata)
- **Chunk** - PDF content chunks (text + metadata)
- **ChatMessage** - RAG conversation history
- **AiSearchHistory** - Image search logs

## Critical Patterns

### 1. **Query Preprocessing Usage**
```typescript
// Always preprocess before embedding for RAG
const preprocessResult = await queryPreprocessorService.preprocessQuery(question);
const embeddings = await Promise.all(
  preprocessResult.simplifiedQueries.map(q => geminiRAGService.generateEmbedding(q))
);

// Use multi-variant search for complex queries
if (isComplexQuery || preprocessResult.confidence < 0.7) {
  // Search with multiple variants, merge results
}
```

### 2. **Error Handling & Retries**
- Gemini API calls use exponential backoff (3 retries)
- Retryable errors: 503, 429, overloaded, quota exceeded
- Always have fallback logic (see `query-preprocessor.service.ts` `basicPreprocessing()`)

### 3. **Logging Convention**
```typescript
console.log(`[ServiceName] Message`);
console.log(`[Chat Stream] Query preprocessing:`, { ... });
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quangtung1412/quizzysmart](https://github.com/quangtung1412/quizzysmart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
