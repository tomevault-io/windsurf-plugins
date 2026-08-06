---
trigger: always_on
description: This document defines how AI agents should behave in the Amorosa AI system. It is written for development, implementation, and future extension of the Messenger assistant, backend services, and admin dashboard.
---

# Agents.md

This document defines how AI agents should behave in the Amorosa AI system. It is written for development, implementation, and future extension of the Messenger assistant, backend services, and admin dashboard.

## 1. Project Overview

Amorosa AI is an AI-powered Facebook Messenger assistant for flower shops. It answers customer questions, recommends products, creates draft orders, books appointments, and escalates complex conversations to humans when needed.

The system is designed around tool calling, retrieval-augmented generation, conversation memory, and integration with business workflows such as Google Calendar and Slack.

## 2. Goals

- Reduce repetitive customer support work.
- Improve response speed and consistency.
- Automate product inquiries, order intake, and appointment booking.
- Keep the experience natural in English and Filipino.
- Escalate to humans when the AI should not decide alone.
- Provide a portfolio-quality example of practical AI engineering.

## 3. Tech Stack

- Frontend: React, Vite, Tailwind CSS, React Router, TanStack Query.
- Backend: FastAPI, LangGraph, LangChain, SQLAlchemy, Alembic.
- AI: OpenRouter, configurable LLM model, tool calling, RAG.
- Database: PostgreSQL with pgvector.
- Integrations: Facebook Messenger Platform, Google Calendar API, Slack API.
- Deployment: Docker, Railway or Render for development, VPS or cloud VM for production.

## 4. Folder Structure

The repository should stay modular and easy to extend.

```text
amorosa-ai/
  frontend/
  backend/
  knowledge/
  migrations/
  docs/
  agents.md
  README.md
  PRD.md
  ARCHITECTURE.md
```

Suggested backend layout:

```text
backend/
  app/
    api/
    core/
    db/
    models/
    services/
    tools/
    workflows/
    rag/
```

Suggested frontend layout:

```text
frontend/
  src/
    components/
    pages/
    hooks/
    api/
    routes/
    styles/
```

## 5. Coding Standards

- Prefer small, focused modules.
- Keep functions readable and easy to test.
- Use explicit types where they improve clarity.
- Avoid deeply nested branching when a helper function is clearer.
- Keep business logic out of route handlers when possible.
- Name functions and files by intent, not implementation detail.
- Add validation at the boundary of each external input.
- Favor deterministic behavior for agent tools and workflows.

## 6. Backend Standards

- Use FastAPI for HTTP endpoints and webhooks.
- Keep request handlers thin and delegate work to services or workflow classes.
- Use SQLAlchemy models and Alembic migrations for schema changes.
- Store conversation, customer, order, and appointment data in PostgreSQL.
- Validate webhook signatures and external payloads before processing.
- Keep secrets in environment variables.
- Use structured logging for every important workflow step.
- Design tools to be idempotent where possible.

## 7. Frontend Standards

- Build the dashboard as an admin-only interface.
- Use React and Vite for a fast developer experience.
- Keep UI state predictable and data access centralized.
- Use TanStack Query for server state.
- Make modules easy to scan: conversations, orders, appointments, products, knowledge base, analytics, and settings.
- Prefer clear information hierarchy over decorative complexity.
- Make mobile behavior functional even if the dashboard is primarily desktop-oriented.

## 8. AI Agent Design

The AI agent should behave like a capable shop assistant, not a generic chatbot.

Core responsibilities:

- Detect customer intent.
- Maintain conversational context.
- Decide when to answer directly and when to call a tool.
- Retrieve knowledge before answering policy, product, or service questions.
- Collect structured data for orders and appointments.
- Escalate when confidence is low or the request is sensitive.

Design principles:

- The agent should not guess on business-critical details.
- Tool results should be treated as authoritative.
- The agent should ask one clear follow-up question at a time when information is missing.
- The assistant should preserve a friendly, concise, service-oriented tone.

## 9. RAG Implementation

RAG should answer questions using markdown-based knowledge documents.

Implementation expectations:

- Store source content in `knowledge/` as markdown files.
- Chunk documents by semantic sections.
- Generate embeddings and store them in pgvector.
- Retrieve the most relevant chunks before composing the final answer.
- Prefer RAG for product details, policies, business hours, delivery rules, and consultation information.
- Reindex knowledge whenever documents change.

Retrieval rules:

- Use the knowledge base before the model improvises on factual business data.
- If retrieval is weak or incomplete, the agent should say so and offer escalation or clarification.

## 10. Database Design

The database should support conversation history, customer records, workflow state, and business objects.

Core tables:

- customers
- conversations
- messages
- products
- orders
- appointments
- knowledge_documents
- knowledge_chunks
- escalations

Design principles:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChadBojelador/customer-operations-agent](https://github.com/ChadBojelador/customer-operations-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
