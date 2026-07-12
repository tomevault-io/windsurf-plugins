---
trigger: always_on
description: - Vor Code-Aenderungen zuerst `ARCHITECTURE.md`, `docs/blocks/STATE.md`, relevante ADRs in `docs/decisions/` und bei Refactors `docs/techdebt.md` lesen; `README.md` enthaelt noch veraltete Status-Passagen, `STATE.md` ist aktueller.
---

# AGENTS.md

## Sources Of Truth
- Vor Code-Aenderungen zuerst `ARCHITECTURE.md`, `docs/blocks/STATE.md`, relevante ADRs in `docs/decisions/` und bei Refactors `docs/techdebt.md` lesen; `README.md` enthaelt noch veraltete Status-Passagen, `STATE.md` ist aktueller.
- Aktueller Stand laut `STATE.md`; neue Features brauchen User-Entscheidung bzw. ADR/Spec-Update vor Code.
- Doc-Sprache und Code-Kommentare Deutsch; Code-Bezeichner und technische Strings Englisch. UI-Sprache ausschliesslich Englisch (ADR-0045); der Sprach-Sweep-Test `tests/test_ui_language.py` schlaegt bei deutschen UI-Strings fehl.

## Architecture Constraints
- Stack ist fix: Python 3.13, Flask, SQLAlchemy 2.x, Alembic, Pydantic v2, Postgres 17, Jinja2/HTMX/Alpine/Plain CSS (eigenes Design-Token-Set). Frontend-Build via esbuild + lightningcss in Docker-Build-Stage (`node:20-alpine`); Production-Image (`python:3.13-slim`) hat keine Node-Runtime (ADR-0032). Tailwind/DaisyUI wurden entfernt.
- LLM-Stack: `openai`-Python-SDK fuer Risk-Reviewer und Per-Group-Chat. `pydantic-ai-slim[openai]` + `trafilatura` ausschliesslich fuer den optionalen `research-worker` (default-off); kein Ersatz fuer den bestehenden LLM-Client.
- fathometer ist Push-only: ueberwachte Server senden Trivy-Rootfs-Scans an `/api/scans`; keine SSH-/Server-Credentials oder Pull-Scanner einbauen.
- Trivy-JSON-Felder nicht erfinden: Pydantic-Schemas an echten Fixtures unter `tests/fixtures/trivy/` orientieren und `extra="ignore"` fuer Forward-Compat beibehalten.
- Keine `|safe`-Ausgabe fuer Client-/LLM-Daten; Markdown/HTML ueber `nh3`, nicht `bleach` oder direktes `markdown`.
- SQLAlchemy-`text()` nur mit gebundenen Parametern; bevorzugt ORM/SQLAlchemy-Ausdruecke.
- Out of scope bleibt hart: Notifications, Multi-User/RBAC/OIDC, Mobile-Optimierung, Container-Image-Scans, Repo-Scans, Misconfig/Secret-UI, PDF, Redis/verteiltes Rate-Limit, SBOM/License-Findings.

## Entrypoints And Services
- App-Factory ist `app:create_app()`; Container-Entrypoint `scripts/entrypoint.sh` wartet auf DB, fuehrt `alembic upgrade head` aus und startet Gunicorn mit `gthread`.
- Compose startet vier Services: `db`, `app`, `fathometer-llm-worker`, `fathometer-research-worker`. Der Research-Worker ist optional, operator-gated, default-off — in Air-Gap-Deploys einfach weglassen.
- `FM_ENCRYPTION_KEY` ist Pflicht; `.env.example` zeigt die Generatoren. `FM_PUBLIC_URL` in Production setzen, sonst rendert `/install.sh` hinter TLS-Proxies falsche interne HTTP-URLs.
- Dashboard-Live-Updates laufen ueber HTMX-Polling, nicht SSE; einziger SSE-Endpunkt ist der Per-Group-Chat-Stream (`GET /servers/<id>/groups/<gid>/chat/stream`).
- Externe Feed-Pulls fuer EPSS/KEV laufen im Worker; Air-Gap-Deploys setzen `FM_FEED_PULL_DISABLED=true`.

## Quality Gate Rules (verbindlich)
- **Erlaubt** sind ausschliesslich drei proactive Gates: (1) Linter (`ruff check . && ruff format --check .`, `shellcheck agent/*.sh`), (2) Static Analyzer (`mypy app/`), (3) Pure-Unit-Tests (`pytest` Default-Selektion).
- **Verboten** ohne ausdrueckliche User-Anweisung pro Lauf: `pytest -m db_integration`, `pytest -m acceptance`, `pytest -m integration`, `pytest -m bench`, `bats`, `RUN_E2E=1`, Docker-Compose-Smoke, Alembic-Roundtrip, Browser-/Playwright-Tests. Keine neuen `.bats`- oder `.sh`-Test-Dateien anlegen ohne User-Genehmigung.
- Jeder `pytest`-Bash-Aufruf hat ein `timeout`-Argument: Default-Suite `timeout: 120000` (2 min), fokussierter Sub-Lauf `timeout: 60000` (1 min). Keine pytest-Aufrufe ohne Timeout.

## Verification Commands
- Standard-Reihenfolge: `ruff check . && ruff format --check .`, dann `mypy app/`, dann fokussierte Tests, dann bei Bedarf Full-Suite.
- Full default suite: `pytest -v` oder mit Coverage-Gate `pytest -v --cov=app --cov-fail-under=85`.
- Adversarial suite separat laufen lassen: `pytest tests/adversarial/ -v`.
- Shell-Agenten pruefen: `shellcheck agent/*.sh`.
- Migration roundtrip und Compose smoke nur bei expliziter User-Anweisung.
- Release-CI baut nur `linux/amd64`; arm64 ist in `.github/workflows/release.yml` bewusst deaktiviert.

## Test Quirks
- `pytest.ini` schliesst `bench`, `integration` und `acceptance` per Default aus; gezielt laufen lassen mit `pytest -m bench`, `pytest -m integration` oder `pytest -m acceptance`.
- Viele DB-Tests nutzen echten Postgres unter `TEST_DATABASE_URL` oder Default `postgresql+psycopg://fathometer:fathometer@localhost:55432/fathometer_test`; ohne erreichbare DB werden sie geskippt.
- Lokale Test-DB: `docker run -d --name fathometer-test-db -e POSTGRES_USER=fathometer -e POSTGRES_PASSWORD=fathometer -e POSTGRES_DB=fathometer_test -p 55432:5432 postgres:17-alpine`.
- Acceptance/Migration-Tests koennen wegen bekannter `tests/conftest.py::_truncate_all`-Race empfindlich sein; siehe `docs/techdebt.md` TD-004 vor RC-Verifikation.
- Worker-Tests, die Mode/Budget mid-test aendern, brauchen den Test-Helper `invalidate_throttle_caches_for_tests()` wegen v0.9.6 Mode-/Budget-Caches.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [THEKROLL-LTD/Fathometer](https://github.com/THEKROLL-LTD/Fathometer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
