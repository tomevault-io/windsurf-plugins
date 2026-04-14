---
trigger: always_on
description: This document provides essential context for AI agents and developers working on this project.
---

# AGENTS.md - Working in the ai_app_challenge Repository

This document provides essential context for AI agents and developers working on this project.

## Project Overview

A **Student Readiness Inventory survey application** built entirely in Gleam:

- **Backend project** (`backend/gleam.toml`): Erlang target, runs on BEAM with Wisp web framework
  - Generates static HTML at build-time from `questions.txt`
  - Serves pre-generated HTML files
  - Receives form submissions and logs to terminal

- **Root project** (`gleam.toml` at root): Shared utilities (JavaScript target for tests)
  - Contains `questions_parser.gleam` used by backend
  - Minimal - mostly for code organization

**Key Feature**: Survey form is **generated once at build-time** from `questions.txt` using Lustre elements, producing static HTML files. No dynamic template rendering at runtime.

## Essential Commands

### Root Project (Shared utilities)

```bash
# From C:/Users/dougl/coding/ai_app_challenge/
gleam build              # Compile shared modules
gleam test               # Run tests (uses gleeunit)
gleam format --check src test  # Check formatting
```

### Backend Project

```bash
# From C:/Users/dougl/coding/ai_app_challenge/backend/
gleam deps download      # Download dependencies
gleam build              # Compile Erlang code
gleam run                # Start the HTTP server on port 8000
gleam test               # Run tests
gleam run -m build       # Generate static HTML from questions.txt
```

### Build & Deploy

```powershell
# From repository root (Windows PowerShell)
.\scripts\build_frontend.ps1
```

This script:
1. **Generates static HTML**: Reads `questions.txt`, runs `gleam run -m build` in backend
   - Generates `backend/priv/static/survey.html` with all questions pre-rendered
   - Generates `backend/priv/static/thanks.html`

Must be run before starting the backend for the first time or after changing `questions.txt`.

### CI/Verification

The GitHub Actions workflow (`.github/workflows/test.yml`) runs:
- `gleam deps download`
- `gleam test`
- `gleam format --check src test`

OTP version: 28, Gleam version: 1.15.2

## Project Structure

```
ai_app_challenge/
├── src/                          # Root project source (shared utilities)
│   ├── ai_app_challenge.gleam   # Shared modules
│   └── questions_parser.gleam   # Question parsing utility (used by backend)
├── test/
│   └── ai_app_challenge_test.gleam      # Tests using gleeunit
├── backend/                      # Backend project (main application)
│   ├── src/
│   │   ├── ai_app_challenge_backend.gleam  # Main entry
│   │   ├── backend_app.gleam           # Server bootstrap & port setup
│   │   ├── build.gleam                 # Build-time HTML generation from questions.txt
│   │   ├── graceful_shutdown.gleam     # Erlang FFI for OTP shutdown
│   │   ├── questions_parser.gleam      # Question parsing (mirrors root/src/)
│   │   └── survey/
│   │       ├── router.gleam            # HTTP request routing
│   │       ├── handlers.gleam          # Route handlers (GET /survey, POST /api/survey)
│   │       └── schema.gleam            # Form decoding & validation
│   ├── priv/static/              # Generated static files
│   │   ├── survey.html           # Pre-generated from questions.txt (build-time)
│   │   └── thanks.html           # Pre-generated thank you page
│   └── manifest.toml             # Dependency lock file
├── scripts/
│   └── build_frontend.ps1        # Build script: generates HTML from questions.txt
├── questions.txt                 # Single source of truth for survey questions
├── gleam.toml                    # Root project config
└── manifest.toml                 # Root project dependency lock
```

## Application Architecture

### Build-Time HTML Generation

```
questions.txt
    ↓
questions_parser.parse()  // Produces List(Question)
    ↓
render_survey_form()      // Creates Lustre Element tree
    ↓
element.to_document_string()  // Converts to HTML string
    ↓
simplifile.write()        // Writes to backend/priv/static/survey.html
```

### HTTP Request Handling

1. **Routing** (`backend/src/survey/router.gleam`):
   - `GET /` → redirect to `/survey`
   - `GET /survey` → serves pre-generated `priv/static/survey.html`
   - `GET /survey/thanks` → serves pre-generated `priv/static/thanks.html`
   - `GET /favicon.ico` → 204 No Content
   - `GET /__quit?token=<token>` → graceful shutdown
   - `POST /api/survey` → form submission handler

2. **Survey Form** (`backend/priv/static/survey.html`, generated at build time):
   - Pure HTML form with all questions pre-rendered
   - No client-side generation or dynamic content
   - Submits as HTTP POST to `/api/survey`
   - Inputs are named by question ID (e.g., `q1_confidence`, `q2_time_management`)

3. **Form Submission** (`backend/src/survey/handlers.gleam` → `post_survey()`):
   - Extracts form values from `application/x-www-form-urlencoded` body
   - Calls `schema.decode()` to parse into `Submission` record
   - Logs to stdout: name, email, grade, answers (dict), notes
   - Redirects to `/survey/thanks` on success, or returns 400 on decode error

4. **Data Model** (`backend/src/survey/schema.gleam`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0xBlue2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
