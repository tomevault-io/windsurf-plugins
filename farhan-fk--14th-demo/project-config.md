---
trigger: always_on
description: This is a **Retrieval-Augmented Generation (RAG) Chatbot** system that answers questions about course materials using semantic search and Claude AI.
---

# CLAUDE.md - AI Context Documentation

## Project Overview
This is a **Retrieval-Augmented Generation (RAG) Chatbot** system that answers questions about course materials using semantic search and Claude AI.

**Repository**: https://github.com/farhan-fk/14th-Demo  
**Original Source**: https://github.com/https-deeplearning-ai/starting-ragchatbot-codebase

---

## Technology Stack

- **Backend Framework**: FastAPI (Python 3.13+)
- **Frontend**: HTML/CSS/JavaScript with Marked.js for markdown rendering
- **Vector Database**: ChromaDB for semantic search and embeddings
- **AI Model**: Anthropic Claude (via API) for response generation
- **Embeddings**: Sentence Transformers for text vectorization
- **Package Manager**: uv (fast Python package installer)

---

## Architecture Overview

### High-Level Components

```
Frontend (Web UI) 
    ↓
FastAPI Server (/api/query, /api/courses)
    ↓
RAG System (Orchestrator)
    ↓
├── Document Processor (Chunking & Parsing)
├── AI Generator (Claude API Integration)
├── Tool Manager (Search Tool)
├── Session Manager (Conversation History)
└── Vector Store (ChromaDB Interface)
    ↓
ChromaDB Collections:
├── course_catalog (metadata)
└── course_content (text chunks)
```

---

## Key Files & Their Responsibilities

### Backend (`backend/`)

| File | Purpose |
|------|---------|
| `app.py` | FastAPI server, defines API endpoints, serves frontend |
| `rag_system.py` | Main orchestrator coordinating all components |
| `vector_store.py` | ChromaDB wrapper for storing/retrieving course chunks |
| `ai_generator.py` | Anthropic Claude API client for response generation |
| `document_processor.py` | Processes course documents into structured chunks |
| `search_tools.py` | Tool-based search system for RAG (used by Claude) |
| `session_manager.py` | Manages conversation history per session |
| `models.py` | Pydantic data models (Course, Lesson, CourseChunk) |
| `config.py` | Configuration settings and environment variables |

### Frontend (`frontend/`)

| File | Purpose |
|------|---------|
| `index.html` | Web interface structure |
| `script.js` | Handles user interactions, API calls, message display |
| `style.css` | UI styling |

### Data (`docs/`)

Contains 4 course transcript files:
- `course1_script.txt` - Building Towards Computer Use with Anthropic
- `course2_script.txt` - MCP: Build Rich-Context AI Apps with Anthropic
- `course3_script.txt` - Advanced Retrieval for AI with Chroma
- `course4_script.txt` - Prompt Compression and Query Optimization

---

## Query Processing Flow

### Step-by-Step User Query Journey

1. **User Input** → Frontend captures query from input field
2. **API Request** → POST to `/api/query` with `{query, session_id}`
3. **RAG System** → Receives request, retrieves conversation history
4. **AI Generator** → Calls Claude API with:
   - System prompt
   - User query
   - Conversation history
   - Available tools (search_course_content)
5. **Tool Execution** (if Claude decides to search):
   - Search Tool → Vector Store
   - ChromaDB performs semantic search
   - Returns top N relevant chunks with metadata
6. **Response Generation** → Claude uses retrieved context to generate answer
7. **Session Update** → Conversation history saved
8. **API Response** → Returns `{answer, sources, session_id}`
9. **Frontend Display** → Shows formatted answer with collapsible sources

---

## Document Processing Pipeline

### Input Format
Course documents follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson N: [lesson title]
Lesson Link: [lesson url]
[lesson content...]
```

### Processing Steps

1. **Metadata Extraction**: Parse first 3-4 lines for course info
2. **Lesson Parsing**: Identify lesson markers (`Lesson N: title`)
3. **Text Chunking**: Split content using sentence-based chunking
   - Configurable chunk size (e.g., 1000 chars)
   - Overlap between chunks for context continuity
   - Preserves sentence boundaries
4. **Context Enhancement**: Prepend metadata to chunks
   - Format: `"Course {title} Lesson {N} content: {chunk}"`
5. **Vectorization**: Generate embeddings using Sentence Transformers
6. **Storage**: Save to ChromaDB with metadata (`course_title`, `lesson_number`, `chunk_index`)

---

## ChromaDB Collections

### 1. `course_catalog` (Metadata)
Stores course-level information for semantic course name matching:
- **ID**: Course title (unique identifier)
- **Document**: Course title text
- **Metadata**: `{title, instructor, course_link, lessons_json, lesson_count}`

### 2. `course_content` (Text Chunks)
Stores searchable course content:
- **ID**: `{course_title}_{chunk_index}`
- **Document**: Chunk text with context
- **Metadata**: `{course_title, lesson_number, chunk_index}`

---

## API Endpoints

### POST `/api/query`
**Purpose**: Process user questions and return AI-generated answers

**Request Body**:
```json
{
  "query": "What is RAG?",
  "session_id": "session_1" // optional
}
```

**Response**:
```json
{
  "answer": "RAG (Retrieval-Augmented Generation) is...",
  "sources": ["Advanced Retrieval for AI - Lesson 2"],
  "session_id": "session_1"
}
```

### GET `/api/courses`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farhan-fk/14th-Demo](https://github.com/farhan-fk/14th-Demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
