---
trigger: always_on
description: This file guides AI agents that create or modify the PaperScout.ai codebase.
---

# CLAUDE.md

# Repository Instructions for AI Coding Agents

This file guides AI agents that create or modify the PaperScout.ai codebase.
PaperScout.ai is a web application that sends daily or weekly updates about
recent literature in a user's research field, summarizes selected articles with
AI, and teaches young researchers how search strategies work.

Before doing any work in this repository, read these files in full when they
exist:

1. `README.md`
2. `PLANNING.md`, `PAPERSCOUT.md`, or the project planning Markdown document
3. `LICENSE`
4. `CODING_STANDARDS.md`
5. `DEVELOPER.md`
6. `QUICKSTART.md`
7. `docs/architecture/` documents
8. `docs/api/` documents
9. `docs/user-guides/` documents

The project planning document is the product source of truth. It defines the
mission, license, functional requirements, non-functional requirements,
technology choices, planned units, testing expectations, and integration tests.

`AGENTS.md` is the execution source of truth for AI agents. It defines how an
agent should plan, scope, implement, test, and document changes.

When instructions conflict, use this order of authority:

1. Explicit user request for the current ticket.
2. This `AGENTS.md` file.
3. `CODING_STANDARDS.md`, if present.
4. The PaperScout.ai planning document.
5. `README.md` and `DEVELOPER.md`.
6. Existing code patterns.

Never remove the project license or copyright notices.

---

## Project Mission

PaperScout.ai helps young researchers stay current with recently published
literature. The system must:

1. Let users register, log in, and manage their profiles securely.
2. Let users create, edit, delete, and review literature search queries.
3. Retrieve relevant articles from PubMed and arXiv.
4. Select the most relevant articles based on search terms, preferences, and
   feedback.
5. Generate concise, readable AI summaries for selected articles.
6. Send scheduled updates by the user's selected delivery method.
7. Store search history and returned article metadata.
8. Ask for optional feedback on result relevance and summary usefulness.
9. Use feedback to improve future search strategies and summarization.
10. Show the search strategy used for each update so users can learn.

The product is for users who may be new to literature search. Favor clear
interfaces, obvious workflows, simple language, and helpful explanations over
feature density.

---

## License Requirements

PaperScout.ai is licensed under the GNU Affero General Public License v3.0 or
later.

All agents must:

1. Preserve AGPL license text and notices.
2. Avoid adding dependencies with licenses that conflict with AGPL-3.0-or-later.
3. Prefer open-source libraries and frameworks.
4. Document any dependency license concerns in the implementation plan.
5. Never replace the license without explicit approval.

---

## Core Technology Stack

Use the stack defined in the planning document unless an approved plan says
otherwise.

### Frontend

1. JavaScript.
2. React.
3. CSS.
4. Jest.
5. React Testing Library.
6. Cypress for browser-level end-to-end tests.

### Backend

1. Python.
2. FastAPI.
3. Pytest.
4. `unittest.mock` for external dependency mocking.
5. Pydantic models for request and response validation.

### Databases

1. SQLite for local development when needed.
2. PostgreSQL for production and integration testing.
3. Database constraints for uniqueness, nullability, and foreign keys.
4. Rollback or isolated database state for every database test.

The planning document mentions Django-style database tests in one section, but
FastAPI is the selected backend framework. Do not add Django unless an approved
plan explicitly changes the backend architecture.

### External Services

1. PubMed E-utilities for biomedical literature retrieval.
2. arXiv API for preprint retrieval.
3. AI summarization provider, configured through environment variables.
4. Postmark for email delivery, if approved and configured.
5. Plivo for SMS delivery, if approved and configured.
6. CronJob-style scheduled jobs for update delivery.

Do not hard-code external service credentials.

---

## Required Project Structure

Use the project structure from the planning document unless an approved plan
changes it. Keep implementation files and tests aligned by unit.

```text
project-root/
├── source/
│   ├── unit-01-landing-page/
│   ├── unit-02-about-app-page/
│   ├── unit-03-about-developer-page/
│   ├── unit-04-sign-up-page/
│   ├── unit-05-login-page/
│   ├── unit-06-user-dashboard/
│   ├── unit-07-user-profile/
│   ├── unit-08-search-history-page/
│   ├── unit-09-new-query-page/
│   ├── unit-10-previous-search-page/
│   ├── unit-21-search-history/
│   ├── unit-22-user-feedback/
│   ├── unit-23-update-notifications/
│   ├── unit-24-pubmed-api/
│   ├── unit-25-arxiv-api/
│   ├── unit-26-article-selection-summarization/
│   ├── unit-27-search-query-optimization/
│   ├── unit-28-update-formatting/
│   ├── unit-29-admin-capabilities/
│   ├── unit-30-access-scheduler-db/
│   ├── unit-31-access-user-db/
│   ├── unit-32-access-search-history-db/
│   ├── unit-33-access-feedback-db/
│   ├── unit-34-update-scheduler-db/
│   ├── unit-35-update-user-db/
│   ├── unit-36-update-search-history-db/
│   ├── unit-37-update-feedback-db/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecelias/PaperScoutDesignDocs](https://github.com/ecelias/PaperScoutDesignDocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
