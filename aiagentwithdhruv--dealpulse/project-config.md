---
trigger: always_on
description: > Last updated: 2026-03-22
---

# CLAUDE.md — DealPulse

> Last updated: 2026-03-22

You are the principal architect and senior software engineer for this repository.

## Project Identity

**Product Name:** DealPulse (Euron Course Project P3)
**GitHub:** `aiagentwithdhruv/dealpulse`
**Subdomain:** dealpulse.aiwithdhruv.com
**Type:** AI-powered real-time sales coaching & lead intelligence SaaS
**MVP Goal:** Ship a functional platform with live call transcription, real-time objection detection, AI coaching suggestions, deal scoring, pipeline analytics, and post-call reports.
**Full Vision:** 16-module sales intelligence engine with CRM sync, conversation intelligence, team performance, and ML-based deal prediction. See `docs/PRD.md`.
**Target Customers:** B2B sales teams, call centers, CRM platforms, sales agencies.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14+, TypeScript, Tailwind CSS |
| Backend | Python 3.11+, FastAPI, Pydantic v2 |
| Database | Supabase (PostgreSQL + pgvector) |
| Cache/Queue | Redis |
| AI (STT) | Deepgram Nova-2 (real-time streaming WebSocket) |
| AI (LLM) | OpenAI API (GPT-4o for reasoning, Haiku for classification) |
| AI (ML) | XGBoost (deal scoring, lead scoring — post-MVP) |
| AI Orchestration | LangGraph (multi-agent, typed state, supervisor) — post-MVP |
| Auth | Supabase Auth + JWT, RBAC |
| Deployment | Docker, AWS (ECS Fargate, ALB, S3, SQS) |

**Cost Targets:** AI coaching suggestion < $0.03 | Objection classification $0.001/segment | Deal scoring $0 (heuristic) | Post-call summary $0.05-$0.10 | Total per call ~$0.50-$0.80

---

## MVP Scope — What to Build NOW

### INCLUDE in MVP (Phase 1)

1. **Auth & RBAC** — Login/signup, role-based access (admin, manager, rep), JWT via Supabase Auth
2. **Call Transcription** — Deepgram streaming, speaker diarization, live transcript display, storage
3. **Live Call Coaching** — WebSocket-based real-time AI suggestions during calls, objection responses
4. **Objection Detection** — Haiku classifier on transcript chunks, 8 base objection types, confidence scoring
5. **Deal Scoring (Heuristic)** — Weighted signal scoring, factor attribution, per-call score updates
6. **Post-Call Reports** — AI summary, key moments, action items, talk-to-listen ratio, sentiment
7. **Sales Rep Dashboard** — Live call view (transcript, objections, suggestions, score), deal pipeline, metrics
8. **Manager Dashboard** — Team monitoring, pipeline health, rep performance, call review queue
9. **Pipeline Analytics** — Visual pipeline (Kanban + table), revenue forecast, velocity metrics, stuck deal alerts
10. **Call Recording** — S3 storage, replay with synchronized transcript
11. **Playbook Management** — Create/manage playbooks, coaching templates, objection response scripts
12. **Admin Panel** — User management, AI config, organization settings
13. **Basic Lead Scoring** — Score based on engagement and intent signals from calls
14. **Notifications** — In-app + email alerts for deal risks and stalled deals

### EXCLUDE from MVP (Post-launch — see docs/PRD.md for full 16 modules)

- CRM integration (Salesforce, HubSpot, Zoho)
- Advanced conversation intelligence (topic extraction, buying signals)
- Team performance dashboards and leaderboards
- Coaching insights and AI-generated recommendations
- Knowledge base RAG (battle cards during calls)
- Slack integration for alerts
- Custom ML scoring models (XGBoost)
- Mobile app (native iOS/Android)
- Churn prediction, multi-language, video calls
- Webhook outbound / SDK / Developer APIs

---

## Key Docs (Read Before Coding)

- `docs/PRD.md` — Product requirements (16 modules), user flows, milestones, pricing
- `docs/ARCHITECTURE.md` — Layers, data flows, AI/ML pipeline, key decisions
- `docs/API_SPEC.md` — 13 API sections, endpoints, schemas, WebSocket events, error responses
- `docs/DB_SCHEMA.md` — 25 tables, indexes, RLS policies, enums, relationship diagram
- `docs/DEPLOYMENT.md` — Docker, AWS ECS, CI/CD, environment variables, monitoring

---

## Architecture Overview

```
Clients: Sales Rep Dashboard + Manager Dashboard + Admin Panel (web)
              |
         ALB / API Gateway (WebSocket + REST)
              |
    +---------+---------+
    |                   |
Next.js App        FastAPI Backend
(Rep + Manager     (REST + WebSocket)
 + Admin UI)            |
    |          Services Layer
    |          (Call, Transcription, Objection,
    |           Coaching, DealScoring, Analytics,
    |           LeadScoring, Playbook, CRM, Auth)
    |                   |
    |          Repositories / Integrations
    |                   |
    +----> Supabase (PostgreSQL + pgvector)
                        |
              Redis (cache, pub/sub, queues)
                        |
              Deepgram (STT) + OpenAI (LLM)
                        |
              AWS S3 (recordings) + SQS (async jobs)
```

**One app, three role-gated views:**
- **Rep routes** (`/dashboard/*`): Live call view, deal pipeline, personal metrics, post-call analytics
- **Manager routes** (`/manager/*`): Team monitoring, pipeline health, rep performance, call review
- **Admin routes** (`/admin/*`): Organization setup, CRM config, user management, AI settings
- Role-based access via middleware; shared component library

---

## LangGraph Agents (5) — Post-MVP


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aiagentwithdhruv/dealpulse](https://github.com/aiagentwithdhruv/dealpulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
