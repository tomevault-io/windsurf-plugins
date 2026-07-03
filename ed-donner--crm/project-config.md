---
trigger: always_on
description: Personal CRM is a simple sales CRM you run on your own computer — think of it as your own private
---

# Personal CRM — Requirements

## Summary

Personal CRM is a simple sales CRM you run on your own computer — think of it as your own private
Salesforce. It helps one person keep track of the companies and people they sell to, the deals in
progress, and the conversations and follow-ups along the way. It runs locally, needs no login, and
works entirely on your machine.

The goal is a clean, focused tool that does the everyday CRM essentials really well: clear lists of
your organizations, contacts and deals; a visual sales pipeline you can drag deals through; a place
to jot notes and track follow-ups; and a dashboard that shows how things are going. It should feel
sharp and professional, and be genuinely pleasant to use.

## Platform

The app has five sections in the main navigation. Organizations, Contacts and Deals each support the
same basics: **add, search, edit and delete**. On first launch the app comes pre-loaded with
realistic sample data, so every screen looks alive immediately.

- **Dashboard** (the landing page) — an at-a-glance view of how sales are going: a chart of deals won
  per month, revenue won per month, a feed of recent activity, and a list of upcoming and overdue
  follow-ups.
- **Organizations** — the companies you do business with. A searchable table to add, edit and remove
  organizations. Click one to see its details, including its contacts and deals.
- **Contacts** — the people you deal with. A searchable table you can also filter by status (lead,
  qualified, customer). Click a contact to see their details and a timeline of their activity. (A
  "lead" is simply a contact with the lead status — there's no separate leads list.)
- **Deals** — the potential sales you're working on. A searchable table showing each deal's stage,
  value and close date. Click a deal to see its details and activity.
- **Pipeline** — your sales pipeline as a visual board: deals shown as cards in columns, one column
  per stage. Drag a deal from one column to another to update its stage. Stages, in order:
  **New → Qualified → Proposal → Negotiation → Won → Lost**.

From any contact or deal you can log an activity (a note, call or email) and optionally give it a
follow-up due date, which then shows up as a task on the dashboard.

## What the CRM keeps track of

Four kinds of record. (Plain-English fields — the exact details are up to the build.)

- **Organization** — a company you do business with. Key info: name, website, industry, and notes.
  An organization has many contacts and many deals.
- **Contact** — a person you deal with. Key info: name, email, phone, job title, the organization
  they belong to, and a status (lead, qualified, or customer).
- **Deal** — a potential sale. Key info: a name, the organization and the main contact it's with, its
  stage in the pipeline, its value in US dollars, and its expected or actual close date.
- **Activity** — something that happened, or needs to happen, with a contact or deal. Key info: type
  (note, call or email), the contact and/or deal it relates to, a description, when it happened, and
  optionally a due date and whether it's done (so the same record doubles as a follow-up task).

## High-level technical guidance

Just enough direction to keep things on track — specific choices are left to the Coding Agent.

- Build it as a single web app using **Vite, React and TypeScript**.
- It runs fully locally and starts with **one simple command**; no accounts, no cloud, no internet
  needed to use it.
- It stores its data **locally on the machine** in a **SQLite** database file.
- **Prefer popular, well-supported libraries over custom code** — for the data tables, the charts,
  and the drag-and-drop pipeline. Don't hand-roll what a mature library does well.
- Keep the implementation simple and conventional. Library, data and structure choices are the
  Coding Agent's call, as long as the requirements and the success criteria below are met.

## Not in scope (v1)

Deliberately left out to keep this small and focused. Do not build these:

- No login, user accounts, multiple users or permissions — it's single-user and local.
- No AI features (these come later).
- No email, calendar or phone integrations.
- Single currency only (US dollars); no multi-currency.
- No reporting or analytics beyond the dashboard described above.
- No tags or custom fields.
- Pipeline stages are fixed (not user-configurable).
- No table pagination, and no data import or export.

## Look and feel

Applies to the whole app:

- Make it **sharp and modern, but still clean and professional**.
- Use the color palette **`#ecad0a` (amber), `#209dd7` (blue) and `#753991` (purple)**, together
  with grays.
- **Avoid** these — they read as generic "AI-generated" tells: background gradients, purple
  backgrounds, buttons with gradients, and panels or cards with a single accent border line down one
  side.

## Phases and success criteria

Build in these phases, in order. **Do not start a phase until every success criterion of the
previous phase is demonstrably met** — each criterion must be something you can actually show
working, not just assert.

### Phase 1 — Running skeleton and data

**Features**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ed-donner/crm](https://github.com/ed-donner/crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
