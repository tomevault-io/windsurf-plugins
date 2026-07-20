---
trigger: always_on
description: Build **ClubLicence Agent**, an AI administrative assistant for French amateur
---

# AGENTS.md

## Project mission

Build **ClubLicence Agent**, an AI administrative assistant for French amateur
basketball clubs.

The assistant helps club volunteers manage player registration and license
paperwork before the season. It reviews a player file, checks the file against a
club-defined license checklist, identifies missing information, assigns a clear
administrative status, drafts a French follow-up email, and updates a simple
club dashboard.

The core story is:

`player file -> checklist -> missing items -> license status -> email draft -> dashboard`

This is an administrative workflow assistant. It is not an official FFBB
submission tool, not a medical document interpreter, not a payment processor,
and not a legal compliance system.

Use only synthetic players, fake parents, fake emails, fake documents, and fake
club data in development, tests, screenshots, the public repository, and the
demo video.

## Capstone constraints

- Target track: **Agents for Good**, because the project helps volunteer-run
  community sports clubs reduce repetitive paperwork.
- Secondary positioning: the same workflow could later apply to sports
  associations, academies, schools, and registration-heavy organizations.
- Final submission deadline: **July 7, 2026 at 2:59 AM EDT**.
- Demonstrate at least three course concepts. This project should visibly show:
  - a multi-agent system implemented with Google ADK for Python;
  - deterministic tools for checklist lookup, file validation, status assignment,
    and dashboard updates;
  - optionally, a small MCP server exposing checklist and communication tools;
  - security and privacy guardrails for minors and medical-adjacent paperwork;
  - reproducible local deployment;
  - Antigravity collaboration in the demo video when required by the course.
- Produce a public repository with a complete `README.md`.
- Prepare a Kaggle writeup of no more than 2,500 words.
- Prepare a public YouTube demo no longer than five minutes.
- Include a cover image and useful screenshots or diagrams.
- Never commit API keys, credentials, real personal data, real medical data,
  real club records, or other secrets.

Optimize decisions for the judging rubric:

1. Clear community problem, credible user value, and meaningful need for agents.
2. A concise story and convincing end-to-end demo.
3. Defensible architecture, implementation quality, and tool use.
4. Pertinent comments explaining non-obvious behavior and safety decisions.
5. Reproducible setup, architecture documentation, and relevant diagrams.

## MVP scope

Prioritize one reliable end-to-end workflow:

1. The club administrator creates or selects a synthetic player file.
2. The intake step records player identity fields, date of birth, category,
   parent or guardian contact when relevant, payment status, received documents,
   and registration deadline.
3. The checklist step selects the required checklist from a club-configured demo
   rules file.
4. The verification step compares received information and documents against the
   required checklist.
5. The status step assigns one clear license status and explains the reason.
6. If required information, documents, or payment are missing, the communication
   step automatically creates a personalized French follow-up email draft.
7. The dashboard step updates club-level counts and highlights files requiring
   action.
8. A human administrator reviews every email draft before anything is sent or
   considered ready to send.

Do not expand the MVP to real FFBB integration, official license submission,
real payment processing, real medical document analysis, authentication,
production database, multi-club management, or unapproved automated
communication with parents.

## Product surfaces

Use a simple Streamlit demo interface unless there is a strong reason to change
it. The dashboard should show:

- total player files;
- complete files;
- incomplete files;
- files missing payment;
- files needing human review;
- urgent files approaching a registration deadline;
- draft emails ready for administrator review;
- per-player status, missing items, recommended next action, and last update;
- concise ADK agent/tool activity events for demo observability.

Do not show hidden chain-of-thought, raw model logs, API keys, secrets, or
unnecessary personal details.

## System architecture

Recommended MVP boundary:

`Streamlit dashboard -> ADK runner -> ADK agents -> deterministic Python tools -> JSON/SQLite state`

Use this split:

- Streamlit owns the visible demo interface.
- Google ADK owns the multi-agent orchestration.
- Deterministic Python tools own business rules and state updates.
- JSON fixtures can be used at first for speed.
- SQLite can replace or supplement JSON when workflow history, approvals, and
  audit logs become useful.

Keep the system runnable locally without external services. If an LLM is used,
make it optional for the core demo path by providing deterministic fallback data
or seeded sample outputs.

## ADK multi-agent design

The minimum agent team is:

- **ClubLicenceOrchestrator**: root ADK agent. It receives the user request,
  coordinates the workflow, passes structured state between specialist agents,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChichiFr/ClubLicenceAgent](https://github.com/ChichiFr/ClubLicenceAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
