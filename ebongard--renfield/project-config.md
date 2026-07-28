---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Renfield is a fully offline-capable, self-hosted **digital assistant** — a personal AI hub for knowledge retrieval, tool access, and smart home control. Serves multiple household users in parallel.

**Tech Stack:** Python 3.11 + FastAPI + SQLAlchemy | React 18 + TypeScript + Vite + Tailwind CSS + PWA | Docker Compose, PostgreSQL 16, Redis 7, Ollama | Satellites: Pi Zero 2 W + ReSpeaker + OpenWakeWord (bare-metal via Ansible). The Esszimmer satellite is the first **arm64 / k8s-pod** satellite — an Orange Pi Zero 3W (Allwinner A733) with a USB ReSpeaker XVF3800, run as a node-pinned privileged pod (`k8s/satellite-esszimmer.yaml`); see the **BLE phone presence** note below.

**LLM:** Local models via Ollama (multi-model: chat, intent, RAG, agent, vision, embeddings).

**Integrations:** Home Assistant, Frigate, n8n, SearXNG, Jellyfin, DLNA, Samsung TV, Paperless, Email, Calendar, Filesystem (watch-folders), Parcel Tracking — all via MCP servers. (Parcel Tracking is `renfield-mcp-tracking`, the `tracking` stdio stanza — multi-carrier with NO third-party aggregator: direct carrier APIs for DHL/Deutsche Post + UPS + FedEx, while DPD/Hermes/GLS have no free public API and return a deep-link into the carrier's own web tracker; the read-only tracking keys are safe in the backend image. DLNA + Samsung TV run as dedicated `hostNetwork` images — `renfield-mcp-dlna` / `renfield-mcp-samsung`; the watch-folder server runs as its own dedicated `streamable_http` deployment — `renfield-mcp-filesystem`, the `files` stanza, holds the SMB/local credentials the backend must not — not in the backend image; the email-mailbox watcher is likewise its own dedicated deployment — `renfield-mcp-email-ingest` (`renfield/email-ingest-mcp`), holding the IMAP credentials the backend must not; all other stdio servers live in the backend image.)

## KRITISCHE REGELN - IMMER BEACHTEN

**NIEMALS `git push` ohne explizite Erlaubnis des Benutzers ausfuehren!** Nach jedem Commit fragen: "Soll ich pushen?" Diese Regel gilt auch nach Session-Komprimierung. Details: `/git-workflow` Skill.

**PR-Lifecycle-Gate: Nach `/review`, VOR dem Merge, IMMER ALLE relevanten Dokumentation aktualisieren.** Kein Nachfragen nötig — das ist Pflicht-Schritt, nicht optional. Sweep statt raten: `grep -rliE "<feature-begriffe>" docs/ README.md CLAUDE.md` und jede betroffene Datei anpassen (typischerweise `CLAUDE.md`, `docs/CIRCLES.md`, `docs/SECOND_BRAIN.md`, `docs/FEATURES.md`). Doc-Update als eigener Commit in denselben PR, dann auf explizite Merge-Freigabe warten. Reihenfolge: `/review` → Docs aktualisieren → warten → merge.

---

## Development Guidelines

### Test-Driven Development (TDD)

**WICHTIG: Bei jeder Code-Aenderung muessen passende Tests mitgeliefert werden.**

1. **Neue API-Endpoints**: Tests in `tests/backend/test_<route>.py` — HTTP status codes, schemas, error handling, edge cases
2. **Neue Services**: Tests in `tests/backend/test_services.py` — unit tests with mocks, `@pytest.mark.unit`
3. **Datenbank-Aenderungen**: Tests in `tests/backend/test_models.py` — model creation, constraints, `@pytest.mark.database`
4. **Frontend-Komponenten**: Tests in `tests/frontend/react/` — RTL rendering, user interactions, MSW API mocks

### Frontend Rules

- **TypeScript only — migration complete.** `src/frontend/src/` is 100% TS (49 .ts + 68 .tsx, 0 .jsx as of v2.4.3). `tests/frontend/react/` migrated alongside. Both have strict mode on with a `npm run typecheck` gate. Type real shapes — no `as any`, no `@ts-nocheck`, no shim files. The "fake-`.tsx` is worse than honest `.jsx`" rule from the W10 migration still applies to any future refactor that can't be typed cleanly in one pass.
- **DESIGN.md is the source of truth.** Before any UI change, read `DESIGN.md` at repo root. Color tokens, fonts, spacing, motion, semantic colors, and the tier visual language are defined there. Do NOT deviate without explicit user approval. In `/review` and `/qa`, flag any code that doesn't match DESIGN.md.
- **Dark Mode**: ALL components must use Tailwind `dark:` variants. Never hardcode colors.
- **i18n**: ALL user-facing strings must use `useTranslation()`. Never hardcode text.
- **Translations**: Add to BOTH `src/frontend/src/i18n/locales/de.json` and `en.json`.
- **Component classes** (in `index.css`): `.card`, `.input`, `.btn-primary`, `.btn-secondary`. New classes per DESIGN.md (e.g., `.tier-badge`, `.atom-row`) must use only DESIGN.md tokens.

## Development Commands

```bash
./bin/start.sh                  # Start entire stack
./bin/update.sh                 # Update system
./bin/debug.sh                  # Debug mode
./bin/quick-update.sh           # Quick backend restart
```

```bash
make lint                       # Lint all (ruff + eslint)
make format-backend             # Format + auto-fix with ruff
make test                       # Run all tests
make test-backend               # Backend tests only
make test-frontend-react        # React component tests (Vitest)
make test-coverage              # Coverage report (fail-under=50%)
```

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ebongard/renfield](https://github.com/ebongard/renfield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
