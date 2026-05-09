---
trigger: always_on
description: A production-ready AI customer support agent that handles WhatsApp, Email (Gmail), and Web Chat — all feeding into one AI brain. Built for a CodeWithMuh YouTube tutorial demonstrating how to replace expensive SaaS tools ($990-$1500/mo) with a self-hosted solution (~$85/mo).
---

# AI Support Agent — Multi-Channel Customer Support System

## Project Overview
A production-ready AI customer support agent that handles WhatsApp, Email (Gmail), and Web Chat — all feeding into one AI brain. Built for a CodeWithMuh YouTube tutorial demonstrating how to replace expensive SaaS tools ($990-$1500/mo) with a self-hosted solution (~$85/mo).

The core principle: **80/20 hybrid approach** — AI handles 80% of routine tickets, humans handle the complex 20%. This is NOT a full replacement for human agents; it's a force multiplier.

## Tech Stack
- **Backend:** Python, Django (Django REST Framework for APIs, Django Channels for WebSockets)
- **AI:** Anthropic Claude API
  - Claude Haiku — ticket classification/routing (fast, cheap, ~$1/M tokens)
  - Claude Sonnet — response generation with RAG context (~$3/M tokens)
- **Database:** PostgreSQL + pgvector (self-hosted via Docker)
- **Channels:**
  - WhatsApp Business Cloud API (Meta)
  - Gmail API (Google Cloud, service account)
  - Custom WebSocket-based web chat widget
- **Dashboard:** Next.js (agent dashboard for escalated tickets + analytics)
- **Infrastructure:** Docker Compose (PostgreSQL, Django backend, Next.js frontend, Redis for Django Channels)
- **Tunneling:** ngrok (for WhatsApp/Gmail webhooks during development)

## Project Structure
```
ai-support-agent/
├── manage.py
├── config/                          # Django project config
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py                      # ASGI entry (Django Channels)
│   └── wsgi.py
├── core/                            # AI brain app
│   ├── models.py                    # Conversation, Message, Ticket, KnowledgeBase models
│   ├── classifier.py                # Haiku-based ticket routing
│   ├── responder.py                 # Sonnet-based response generation
│   ├── knowledge_base.py            # pgvector RAG retrieval
│   ├── embeddings.py                # Text chunking & embedding generation
│   ├── guardrails.py                # Anti-hallucination checks
│   ├── serializers.py               # DRF serializers
│   ├── urls.py
│   └── admin.py
├── channels_app/                    # Multi-channel integration app
│   ├── models.py
│   ├── views.py                     # Webhook endpoints (WhatsApp, Email)
│   ├── consumers.py                 # Django Channels WebSocket consumer (web chat)
│   ├── routing.py                   # WebSocket URL routing
│   ├── unified.py                   # UnifiedMessage normalization
│   ├── whatsapp.py                  # WhatsApp Cloud API helpers
│   ├── email_handler.py             # Gmail API polling & reply
│   ├── serializers.py
│   └── urls.py
├── escalation/                      # Escalation & handoff app
│   ├── models.py
│   ├── detector.py                  # Escalation trigger logic
│   ├── handoff.py                   # Context packaging for human agents
│   ├── sentiment.py                 # Sentiment analysis helper
│   ├── views.py
│   ├── serializers.py
│   └── urls.py
├── dashboard/                       # Next.js agent dashboard
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx             # Dashboard overview
│   │   │   ├── tickets/             # Ticket queue & detail views
│   │   │   └── analytics/           # Analytics panel
│   │   └── components/
│   │       ├── TicketQueue.tsx
│   │       ├── ConversationThread.tsx
│   │       ├── AISidebar.tsx        # Classification + suggested response
│   │       └── ChatWidget.tsx       # Embeddable web chat widget
│   └── package.json
├── scripts/
│   └── seed_knowledge_base.py       # Load FAQ/docs into knowledge base
├── tests/
├── docker-compose.yml               # PostgreSQL, Django, Next.js, Redis, (ngrok optional)
├── Dockerfile                       # Django backend
├── dashboard/Dockerfile             # Next.js frontend
├── .env.example
├── requirements.txt
├── CLAUDE.md
└── README.md
```

## Architecture Flow
```
Customer Message (WhatsApp / Email / Web Chat)
    ↓
Django Backend → Normalize to UnifiedMessage
    ↓
Claude Haiku → Classify (billing | technical | account | complaint | escalate)
    ↓                                    ↓ (confidence < 0.7 OR angry OR asks for human)
    ↓                                    → ESCALATE → Dashboard (human agent)
    ↓
PostgreSQL pgvector → Retrieve relevant knowledge base chunks
    ↓
Claude Sonnet → Generate response (constrained to KB context only)
    ↓
Guardrail Check → Verify no hallucinated policies/prices/guarantees
    ↓
Send Response via original channel
    ↓
Store in PostgreSQL (conversations + messages tables)
```

## Key Design Decisions

### Ticket Classification (Haiku)
- Uses Anthropic's official XML tag pattern for structured output
- Categories: `billing`, `technical`, `account`, `complaint`, `escalate`
- Returns confidence score (0.0-1.0) — auto-escalate below 0.7

### Response Generation (Sonnet)
- RAG-based: only answers from knowledge base context
- System prompt explicitly prohibits answering without KB match
- If no relevant chunks found → graceful decline + offer human handoff


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codewithmuh/ai-support-agent](https://github.com/codewithmuh/ai-support-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
