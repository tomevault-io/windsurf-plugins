---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TalentDrop (职遇) is an AI-powered talent matching platform inspired by Date Drop's deep-matching algorithm. Instead of endless job applications, both candidates and companies complete deep questionnaires (Career DNA / Company DNA), and the algorithm "drops" matched opportunities weekly. The project has a **working full-stack Demo** with dual-side flows, AI chat, resume parsing, knowledge graph visualization, and an interactive theory presentation page.

## Repository Structure

```
vibe-hiring/
├── backend/                    # Python FastAPI backend
│   └── src/
│       ├── main.py             # App entry, router registration
│       ├── seed.py             # Database seed script
│       ├── api/                # REST routes
│       │   ├── auth.py         # JWT authentication
│       │   ├── answers.py      # Questionnaire answer submission
│       │   ├── questions.py    # Question bank retrieval
│       │   ├── company.py      # Company endpoints
│       │   ├── roles.py        # Open position CRUD
│       │   ├── matching.py     # Run matching + dual-action accept/pass
│       │   ├── drop.py         # Weekly drop (candidate + company side)
│       │   ├── scores.py       # DNA score retrieval
│       │   ├── chat.py         # AI conversational profiling
│       │   ├── resume.py       # Resume upload + AI parsing
│       │   ├── profile.py      # User profile CRUD
│       │   └── graph.py        # Knowledge graph + pipeline visualization data
│       ├── models/             # Pydantic models
│       │   ├── database.py     # SQLite schema (all tables)
│       │   ├── user.py, company.py, questionnaire.py, dna_score.py
│       │   ├── match.py        # Dual-action match + drop models
│       │   ├── role.py         # Open position models
│       │   ├── profile.py      # User profile models
│       │   └── chat.py         # Chat message models
│       ├── services/           # Business logic
│       │   ├── scoring.py      # Career DNA scoring
│       │   ├── company_scoring.py, aggregation.py, cas.py
│       │   ├── matching.py     # L1-L2 matching engine (candidate × role)
│       │   ├── drop.py         # Dual drop generation (candidate + company)
│       │   ├── report.py       # Template report (Phase 1)
│       │   ├── llm_report.py   # LLM-powered report (Phase 2)
│       │   ├── chat_service.py # AI chat + entity extraction
│       │   ├── resume_service.py # PDF extraction + AI parsing
│       │   └── graph_service.py  # Knowledge graph data builder
│       ├── data/               # Question banks + seed data
│       └── core/               # Config, deps, logging, middleware
├── frontend/                   # Next.js 16 + React 19 + Tailwind CSS 4
│   └── src/
│       ├── app/
│       │   ├── page.tsx        # Landing / login
│       │   ├── demo/page.tsx   # Interactive theory presentation
│       │   ├── candidate/      # Candidate-side pages
│       │   │   ├── dashboard/, questionnaire/, drop/, chat/, profile/
│       │   │   └── match/[id]/ # Match report + visualization tabs
│       │   └── company/        # Company-side pages
│       │       ├── dashboard/, questionnaire/, candidates/, invite/
│       │       ├── roles/      # Open position management
│       │       ├── drop/       # Company weekly drop
│       │       └── match/[id]/ # Company match detail
│       ├── components/
│       │   ├── layout/         # Header, Sidebar, PageContainer
│       │   ├── ui/             # Button, Card, Input, Modal, Badge, Progress
│       │   ├── questionnaire/  # ProgressHeader, QuestionCard, RankQuestion, BudgetQuestion
│       │   ├── charts/         # RadarChart, KnowledgeGraph, MatchFunnel, DimensionCompare
│       │   └── chat/           # ChatWindow, ChatMessage
│       ├── hooks/              # useAuth, useQuestionnaire
│       └── lib/                # api.ts, types.ts, constants.ts
├── discuss/                    # Product design docs (Chinese)
├── scripts/                    # Dev/deploy shell scripts
└── docker-compose.yml
```

## Key Product Concepts

- **Career DNA**: 8 core dimensions (Pace, Collab, Decision, Expression, Unc, Growth, Motiv, Execution), all spectrums with no "right answer"
- **Weekly Drop**: Tuesday 9PM curated match reveal with compatibility reports
- **Dual Discovery**: Candidates receive matched roles; companies receive matched candidates per role. Both sides Accept/Pass → Mutual Match
- **Three-layer question architecture**: Platform standard (30q, 60%) → Job-type specific (15q, 25%) → Company custom (≤5q, 15%)
- **Matching engine**: L1 hard filter (skills/location) → L2 DNA compatibility (8-dim vector distance × consistency)

## Current Implementation Status

| Feature | Status | Notes |
|---------|--------|-------|
| Career DNA questionnaire | ✅ | 30 questions, 3 question types |
| Company DNA questionnaire | ✅ | 20 questions, CAS scoring |
| L1-L2 matching engine | ✅ | Candidate × Role matching |
| Dual-side Weekly Drop | ✅ | Candidate + Company drops |
| Mutual Match state machine | ✅ | pending → accepted → mutual / passed |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Teejay-first/talentdrop](https://github.com/Teejay-first/talentdrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
