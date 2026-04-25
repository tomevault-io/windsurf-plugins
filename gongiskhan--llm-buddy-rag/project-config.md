---
trigger: always_on
description: You are working on LLM Buddy RAG, a sophisticated multimodal Retrieval-Augmented Generation system that handles both text and visual content. This is a performance-critical module designed to operate both as a library and as an API service.
---

# Claude Code Session Guidelines - LLM Buddy RAG

## Project Overview

You are working on LLM Buddy RAG, a sophisticated multimodal Retrieval-Augmented Generation system that handles both text and visual content. This is a performance-critical module designed to operate both as a library and as an API service.

## Critical Instructions

**MANDATORY**: At the start of every new conversation:
1. Always read PLANNING.md to understand the technical architecture and current implementation status
2. Check TASKS.md to see what work needs to be done
3. Before starting any task, mark it as "in progress" in TASKS.md
4. Upon completing any task, immediately mark it as "DONE" in TASKS.md
5. Add any newly discovered tasks or requirements to TASKS.md as you find them

## Coding Standards

### TypeScript Requirements
- Use TypeScript strict mode for all code
- Define explicit types for all function parameters and return values
- Use interfaces for data structures, types for unions and primitives
- Implement Zod schemas for runtime validation of external data
- Export types from a centralized `types` module

### Code Organization
```typescript
// Preferred structure for modules
export interface ServiceConfig {
  // Configuration interface
}

export class Service {
  private config: ServiceConfig;
  
  constructor(config: ServiceConfig) {
    this.config = config;
  }
  
  // Public methods with clear return types
  async process(input: Input): Promise<Output> {
    // Implementation
  }
}
```

### Performance Guidelines
- Use async/await for all I/O operations
- Implement connection pooling for database and API connections
- Use streaming for large data processing
- Implement proper error boundaries and recovery
- Add performance logging for operations >100ms

### Error Handling
```typescript
// Use custom error classes
export class RAGError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = 'RAGError';
  }
}

// Wrap external calls
try {
  const result = await externalAPI.call();
  return result;
} catch (error) {
  throw new RAGError(
    'External API failed',
    'EXTERNAL_API_ERROR',
    503
  );
}
```

## Architecture Patterns

### Dual-Mode Operation
The system must support both library and API modes:

```typescript
// Library mode - direct usage
import { RAGEngine } from '@llm-buddy/rag';
const rag = new RAGEngine(config);
const results = await rag.search(query);

// API mode - HTTP interface
POST /api/search
{
  "query": "...",
  "options": {...}
}
```

### Dependency Injection
Use dependency injection for testability:

```typescript
export class SearchService {
  constructor(
    private vectorDB: VectorDatabase,
    private metadataDB: MetadataDatabase,
    private embedder: EmbeddingService
  ) {}
}
```

### Repository Pattern
Implement repositories for data access:

```typescript
export interface ContentRepository {
  save(content: Content): Promise<string>;
  findById(id: string): Promise<Content | null>;
  search(criteria: SearchCriteria): Promise<Content[]>;
  delete(id: string): Promise<boolean>;
}
```

## Testing Requirements

### Test Coverage
- Minimum 80% code coverage
- Unit tests for all business logic
- Integration tests for API endpoints
- Performance tests for critical paths

### Test Structure
```typescript
describe('SearchService', () => {
  let service: SearchService;
  let mockVectorDB: jest.Mocked<VectorDatabase>;
  
  beforeEach(() => {
    mockVectorDB = createMockVectorDB();
    service = new SearchService(mockVectorDB);
  });
  
  describe('search', () => {
    it('should return relevant results', async () => {
      // Test implementation
    });
  });
});
```

## API Design Principles

### RESTful Conventions
- Use proper HTTP methods (GET, POST, PUT, DELETE)
- Return appropriate status codes
- Include pagination for list endpoints
- Version the API from the start (/api/v1/)

### Request/Response Format
```typescript
// Consistent response structure
interface APIResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  metadata?: {
    timestamp: string;
    requestId: string;
    version: string;
  };
}
```

### Validation
```typescript
// Use Zod for request validation
const SearchRequestSchema = z.object({
  query: z.string().min(1).max(1000),
  limit: z.number().int().min(1).max(100).default(10),
  offset: z.number().int().min(0).default(0),
  filters: z.object({
    type: z.enum(['text', 'image']).optional(),
    dateRange: z.object({
      from: z.string().datetime().optional(),
      to: z.string().datetime().optional()
    }).optional()
  }).optional()
});
```

## Database Patterns

### Vector Database (FAISS)
- Initialize with appropriate index type (IVF, HNSW)
- Implement periodic index optimization
- Handle index persistence and recovery
- Monitor memory usage

### Metadata Database (SQLite)
```sql
-- Use proper schema design
CREATE TABLE content (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL CHECK(type IN ('text', 'image')),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gongiskhan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
