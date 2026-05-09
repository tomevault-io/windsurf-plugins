---
trigger: always_on
description: You are working on the **Regulatory Intelligence Assistant for Public Service**, an AI-powered platform that helps public servants and citizens navigate complex laws, policies, and regulations with consistent interpretation and reduced cognitive load.
---

# CLAUDE.md - Regulatory Intelligence Assistant

## Project Context

You are working on the **Regulatory Intelligence Assistant for Public Service**, an AI-powered platform that helps public servants and citizens navigate complex laws, policies, and regulations with consistent interpretation and reduced cognitive load.

## Project Goals

- **G7 GovAI Challenge**: Statement 2 - Navigating Complex Regulations
- **Mission**: Streamline interpretation and application of rules to increase consistency and compliance
- **Timeline**: 2-week MVP for competition (Nov 17 - Dec 1, 2025)
- **Target Impact**: 60-75% reduction in research time, 80% reduction in decision inconsistencies

## Key Architecture Components

### 1. Regulatory Knowledge Graph

- Structured representation of legislation, regulations, policies
- Semantic relationships (references, amendments, dependencies, conflicts)
- Cross-jurisdictional mapping
- Program and service connections
- Temporal versioning for changes

### 2. Legal NLP Engine

- Query parsing and entity extraction
- Intent classification (search, compliance, interpretation)
- Named entity recognition (programs, jurisdictions, person types)
- Context enrichment for better understanding
- Plain language generation

### 3. RAG System with Gemini API

- Upload regulatory documents to Gemini
- Semantic search across legal corpus
- Answer questions with source citations
- Handle ambiguous queries with clarification
- Multi-document reasoning

### 4. Semantic Search Engine

- Natural language queries
- Hybrid search (keyword + semantic + graph)
- Faceted filtering and relevance ranking
- Citation-aware results
- Similar regulation discovery

### 5. Compliance Checking Engine

- Form validation against regulations
- Requirement extraction and checking
- Real-time compliance feedback
- Gap analysis for missing information
- Confidence scoring

### 6. Guided Workflows

- Step-by-step processes for common scenarios
- Decision trees for complex regulations
- Dynamic questionnaires
- Pre-filled templates
- Escalation triggers for edge cases

## Technology Stack

- **Frontend**: React with accessible UI
- **Backend**: Python FastAPI for legal processing
- **AI**: Fine-tuned BERT/RoBERTa, Gemini API for RAG
- **Knowledge Graph**: Neo4j 5.15 (custom Docker image with APOC + GDS plugins)
- **Search**: Elasticsearch + Pinecone/Weaviate for vectors
- **Database**: PostgreSQL (metadata), Redis (caching)

## Docker Infrastructure

### Neo4j Configuration

The project uses a custom Neo4j Docker image located at `backend/neo4j/`:

- **Base Image**: `neo4j:5.15-community`
- **Pre-installed Plugins**: APOC (from labs), Graph Data Science 2.6.9
- **Custom Entrypoint**: `docker-entrypoint-wrapper.sh` handles:
  - Stale PID file cleanup on container restart
  - Skipping password setup on subsequent starts
  - Graceful restart without data loss

### Key Docker Commands

```bash
# Start all services
docker compose up -d

# Rebuild Neo4j after Dockerfile changes
docker compose build neo4j --no-cache
docker compose up -d neo4j

# Safe restart (data preserved)
docker compose restart neo4j

# Full reset (WARNING: deletes all data)
docker compose down
docker volume rm regulatory-intelligence-assistant_neo4j_data
docker compose up -d
```

## Critical Requirements

### Legal Accuracy

- All interpretations must cite authoritative sources
- Confidence scores for every recommendation
- Alternative interpretations for ambiguous cases
- Expert validation for high-stakes decisions
- Complete audit trails

### Source Authority

- Only official government legal sources
- Cryptographic verification of content
- Version tracking for all regulations
- Legislative change monitoring
- Precedent database maintenance

### Explainability

- Cite specific sections, clauses, subsections
- Show reasoning chain with legal references
- Flag regulatory conflicts
- Provide confidence levels
- Explain uncertainty clearly

### Performance

- Search response <3 seconds (p95)
- Q&A response <5 seconds (p95)
- Support 100+ concurrent users
- 99.95% uptime
- Process 500-1000 queries daily

## Development Approach

### AI-TDD Process

1. Create expert-labeled test dataset
2. Establish accuracy baselines
3. Implement with continuous validation
4. Legal expert review of outputs
5. Iterative refinement with feedback

### Legal Processing Pipeline

```
User Query → NLP Processing → Entity Extraction →
Intent Classification → Knowledge Graph Query →
Document Retrieval → RAG Generation → Citation Verification →
Confidence Scoring → Response with Sources
```

### Testing Strategy

- Expert-validated test queries (20-30 scenarios)
- Precision/recall testing on legal Q&A
- Citation accuracy verification (100% requirement)
- Cross-validation with legal experts
- User acceptance testing with caseworkers

## Common Scenarios to Handle

### Regulatory Search Query

```
User: "Can a temporary resident apply for employment insurance?"
- Extract entities: person type (temporary resident), program (EI)
- Identify intent: eligibility question

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samjd-zz/regulatory-intelligence-assistant](https://github.com/samjd-zz/regulatory-intelligence-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
