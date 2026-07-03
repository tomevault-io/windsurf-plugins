---
trigger: always_on
description: This file is the entry point for Claude Code (and any AI coding assistant) working on this repository.
---

# ShipFlow — Claude Code Guide

This file is the entry point for Claude Code (and any AI coding assistant) working on this repository.
Read it before touching any code.

---

## Current Milestone: v1.10.0 — "Mobile PWA"

**Current version**: v1.9.0 (released 2026-07-01)
**All PRs target**: `main` branch

ShipFlow is **methodology-agnostic** — supports Shape Up + Kanban + Scrum (v1.1.0 shipped). Competitor migration tooling (v1.2.0) shipped. MCP Server Admin & API Keys (v1.3.0) shipped. Enterprise Auth & UX Depth (v1.4.0) shipped. AI Copilot v2 — AI Pitch Writer, Retrospective Summarizer, Proactive Dashboard Insights (v1.5.0) shipped. MCP Ecosystem (v1.6.0) shipped. Workflow Automations — trigger/action engine, 20 templates, full UI (v1.7.0) shipped. Custom Fields & Advanced RBAC, Wiki / Docs Space, and pluggable Object Storage (v1.8.0) shipped. Production-Grade Self-Hosting — Helm chart, OpenTelemetry, Grafana, audit export, and air-gapped AI mode (v1.9.0) shipped. Next: Mobile PWA — offline support, responsive audit, Web Push, biometric auth (v1.10.0).

### v1.3.0 session map

| Session | Task | Status |
|---------|------|--------|
| S31 | MCP server admin toggle + API key management UI — DB-backed runtime toggle, write-tools toggle, API key CRUD with scopes/expiry, admin oversight | ✅ done |

### v1.2.0 session map

| Session | Task | Status |
|---------|------|--------|
| S27 | CSV import backend — `ImportJob` entity, `CsvImportService` (Jira/Linear/Asana/Generic), `ImportController` | ✅ done |
| S28 | CSV import frontend — stepper UI (Upload → Preview → Done), import history page | ✅ done |
| S29 | Linear API import — OAuth2 flow, GraphQL fetch of issues/cycles/projects | ✅ done |
| S30 | Jira API import — Atlassian OAuth 2.0 (3LO), REST API fetch of issues/sprints/epics | ✅ done |

### v0.8.0 session map (sessions S01–S13)

| Session | Task | Status |
|---------|------|--------|
| S01 | Public roadmap page (`/roadmap`) | ✅ done |
| S02 | Demo seed data refresh | ✅ done |
| S03 | Version alignment (pom.xml 0.8.0, java 21, CORS fix) | ✅ done |
| S04 | Spring Boot upgrade 3.2.1 → 3.4.x | ✅ done |
| S05 | Rate limiting (Bucket4j) + CSP headers + startup secret validation | ✅ done |
| S06 | Docker GHCR CI/CD + React.lazy code splitting | ✅ done |
| S07 | File attachments on tasks — backend | ✅ done |
| S08 | File attachments on tasks — frontend | ✅ done |
| S09 | Bulk task operations — backend | ✅ done |
| S10 | Bulk task operations — frontend | ✅ done |
| S11 | @mention triggers notification | ✅ done |
| S12 | CSV export for task backlog | ✅ done |
| S13 | Interactive onboarding tour (wire TourContext + driver.js) | ✅ done |
| S13.1 | Q&A / RAG hardening — entity disambiguation, multi-turn memory, cache isolation | ✅ done |

**v0.8.0 released: 2026-04-05** ✅

### v0.9.0 session map (sessions S14–S26)

| Session | Task | Status |
|---------|------|--------|
| S14 | Saved Filter Views — backend | ✅ done |
| S15 | Saved Filter Views — frontend | ✅ done |
| S16 | Real-time notifications via SSE | ✅ done |
| S17 | Email notifications (SMTP) | ✅ done |
| S18 | MCP Phase 2 write tools | ✅ done |
| S19 | Playwright E2E — auth flow | ✅ done |
| S20 | Playwright E2E — project management flow | ✅ done |
| S21 | Playwright E2E — pitch lifecycle | ✅ done |
| S22 | Playwright E2E — hill chart | ✅ done |
| S23 | Playwright E2E — task management | ✅ done |
| S24 | BacklogPage decomposition | ✅ done |
| S25 | OrganizationSettings decomposition | ✅ done |
| S26 | PitchDetail decomposition | ✅ done |

**v0.9.0 released: 2026-04-14** ✅

### v1.4.0 session map

| Session | Task | Status |
|---------|------|--------|
| S32 | SSO backend — Spring Security SAML2 + OIDC, IdentityProvider entity, Flyway migration | ✅ done |
| S33 | SSO frontend — identity provider config UI in Org Settings, SSO enforcement toggle, login page SSO button | ✅ done |
| S34 | SCIM 2.0 user provisioning — backend endpoint + frontend toggle | ✅ done |
| S35 | Roadmap interactivity — drag-to-move/resize Gantt bars, date range validation, progress indicators | ✅ done |
| S36 | UX polish — inline pitch title editing, retrospective templates (Went Well/Improve/Action Items), i18n interpolation sweep | ✅ done |
| S37 | Navigation hardening — deep-link routing for all sidebar routes, keyboard shortcut cheat sheet overlay | ✅ done |

**v1.4.0 released: 2026-06-07** ✅

### v1.5.0 session map

| Session | Task | Status |
|---------|------|--------|
| S38 | AI Pitch Writer — LLM-powered Shape Up pitch draft from one-sentence problem description, pre-fills pitch form | ✅ done |
| S39 | Retrospective Summarizer — AI-generated cycle retro summary, template fallback, RetroSummaryPanel below board | ✅ done |
| S40 | Proactive Dashboard Insights — DashboardInsightsPanel (overdue pitches, at-risk cycles, scope creep, velocity), Redis cache | ✅ done |

**v1.5.0 released: 2026-06-07** ✅

### v1.6.0 session map

| Session | Task | Status |
|---------|------|--------|
| S41 | Agentic MCP write tools — `update_task` + `update_pitch` (PATCH-semantic full-field MCP write tools) | ✅ done |
| S42 | Plugin SDK — Maven archetype, plugin registry, first-party plugin scaffold | ✅ done |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farzad-sedaghatbin/ShipFlow](https://github.com/farzad-sedaghatbin/ShipFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
