---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Meu Guia do Super** is a mobile-first grocery store app with indoor wayfinding and navigation. Core user value: shoppers can search for products and get turn-by-turn navigation guidance to locate items inside the physical store. [MappedIn](https://developer.mappedin.com/docs/overview) (specifically their [grocery store demo](https://app.mappedin.com/map/6679882a8298d5000b85ee89?floor=m_f62f718116360827)) is the visual and interaction benchmark for all path-finding screens.

---

## Monorepo Structure

This repo uses a **multi-agent orchestration model**. Each subdirectory has its own CLAUDE.md defining a specialized agent's role and constraints.

```
meu-guia-do-super-v2/
├── .claude/
│   └── commands/             # Project slash command skills (auto-loaded in Claude Code sessions)
│       ├── map-coordinate-transformer.md  # /map-coordinate-transformer → Layout Parser Agent
│       ├── pathfinder-dijkstra-calc.md    # /pathfinder-dijkstra-calc → Routing Logic Agent
│       └── waypoint-rn-ui.md             # /waypoint-rn-ui → UI Generator Agent
├── agents/
│   ├── product_management/   # PM/PO agent — user stories, backlog, acceptance criteria
│   ├── ux_design/            # UX agent — flows, tokens, mobile-first specs, SPECS/ folder
│   ├── quality_assurance/    # QA agent — test plans, Arrange-Act-Assert scripts
│   ├── devsecops/            # DevSecOps agent — CI/CD, secrets, mobile distribution
│   └── wayfinding/           # Wayfinding domain agent cluster
│       ├── layout_parser/    # Coordinate ingestion → SQLAlchemy schema models
│       ├── routing_logic/    # Dijkstra/A* pathfinding via networkx, MappedIn benchmark
│       └── ui_generator/     # React Native wayfinding components, MappedIn visual parity
├── scripts/                  # Python validation gate scripts (CI-integrated, pytest-tested)
│   ├── validate_api_contract.py      # Validates server/api-spec.md completeness
│   ├── validate_agent_handoffs.py    # Validates handoff artifacts between agent stages
│   └── verify_navigation_graph.py   # Validates navigation graph integrity (no floating nodes)
├── tests/                    # Pytest test suite for validation scripts
├── client/                   # React Native / Expo frontend
│   └── src/
│       ├── assets/           # App logo + full-page screenshots for UX reference
│       ├── components/
│       ├── pages/
│       ├── services/         # API clients — must match server/api-spec.md exactly
│       ├── stores/           # Zustand state
│       └── types/
├── server/                   # Python 3.11+ + FastAPI backend
│   ├── api-spec.md           # Single source of truth for all API contracts (v1)
│   └── src/                  # controllers/, repositories/, routes/, services/, utils/
├── ARCHITECTURE.md           # Directory map and folder conventions
├── DESIGN.md                 # Full Starbucks-inspired design system (colors, tokens, components)
└── agents/ux_design/SPECS/   # UI spec packs per flow: ADMIN/, CLIENT/, LANDING/
```

---

## Tech Stack

| Layer               | Technology                                                              |
| ------------------- | ----------------------------------------------------------------------- |
| Mobile              | React Native + Expo                                                     |
| Styling             | NativeWind (Tailwind CSS for RN)                                        |
| State               | Zustand                                                                 |
| Data fetching       | TanStack Query (React Query) with offline caching                       |
| Backend runtime     | Python 3.11+ + FastAPI                                                  |
| Validation          | Pydantic v2 (all incoming inputs treated as hostile; parsed through Pydantic models) |
| ORM                 | SQLAlchemy 2.0 (async) + Alembic migrations                             |
| Database            | Supabase (PostgreSQL)                                                   |
| Auth                | JWT / OAuth2 with role-based access control for admin routes            |
| Caching             | Redis                                                                   |
| Email               | Resend (contact form submissions → Fabio's personal email)              |
| CI/CD               | GitHub Actions                                                          |
| IaC                 | Terraform / OpenTofu                                                    |
| Containerization    | Docker                                                                  |
| Security scanners   | Trufflehog, Snyk, SonarQube                                             |
| Mobile distribution | Fastlane, TestFlight, Google Play Console                               |

---

## Common Commands

The project is in early scaffolding phase. Commands will be added here as `client/` and `server/` are built out.

**Expected server pattern (Python + FastAPI):**

```bash
cd server
uvicorn src.main:app --reload   # FastAPI hot reload

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabalvesfr/meu-guia-do-super-v2](https://github.com/fabalvesfr/meu-guia-do-super-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
