---
trigger: always_on
description: - **Never commit unless explicitly told to.** Wait for the user to ask you to commit changes.
---

# DailyNotes Architecture Guide

## Important Rules

- **Never commit unless explicitly told to.** Wait for the user to ask you to commit changes.

## Project Overview

DailyNotes is a self-hosted daily task and note-taking application that combines the experience of a physical planner with modern web technology. It supports markdown with GitHub Flavored Markdown (GFM) task lists, making it ideal for daily journaling, task tracking, and note management.

**Version Format:** `YYYY.MM.DD-##` (date-based with daily build number, managed by CI)

## Technology Stack

### Backend

- **Framework:** Quart (async Python microframework, Flask-compatible)
- **Database ORM:** SQLAlchemy (raw, without Flask-SQLAlchemy)
- **Authentication:** JWT (JSON Web Tokens) via PyJWT with custom decorators
- **Password Hashing:** Argon2 via argon2-cffi
- **Database Migrations:** Alembic
- **Production Server:** Uvicorn (ASGI)
- **HTTP Client:** httpx (async HTTP requests)
- **Data Encryption:** PyCryptodome (AES encryption for sensitive data at rest)
- **Markdown Processing:** python-frontmatter (for parsing YAML frontmatter)

**Python Version:** Python 3.8+

### Frontend

- **Framework:** Vue.js 2.6
- **Language:** TypeScript 3.5
- **UI Library:** Buefy (Vue wrapper for Bulma CSS)
- **CSS Framework:** Bulma with Bulmaswatch theme (Minty)
- **Editor:** CodeMirror 5 (for markdown editing with syntax highlighting)
- **Markdown Rendering:** Marked (for HTML preview with GFM support)
- **Routing:** Vue Router 3
- **HTTP Client:** Axios with JWT interceptors
- **Utilities:** date-fns, Lodash, Vue Masonry CSS
- **Icons:** FontAwesome Free
- **Fonts:** Fira Code (with ligatures), Montserrat

**Node Version:** 12+

## Project Structure

```
DailyNotes/
├── app/                          # Backend Python application
│   ├── __init__.py              # Quart app initialization
│   ├── routes.py                # API endpoints
│   ├── models.py                # Database models (User, Note, Meta)
│   └── model_types.py           # Custom SQLAlchemy types (GUID)
├── client/                       # Frontend Vue.js application
│   ├── src/
│   │   ├── main.ts              # Vue app entry point
│   │   ├── App.vue              # Root Vue component
│   │   ├── interfaces.ts        # TypeScript interfaces (INote, IHeaderOptions, IMeta)
│   │   ├── router/
│   │   │   └── index.ts         # Vue Router configuration & auth guards
│   │   ├── services/            # API service layer
│   │   │   ├── requests.ts      # Axios configuration with JWT interceptors
│   │   │   ├── notes.ts         # Note/day API service methods
│   │   │   ├── user.ts          # User auth service
│   │   │   ├── sidebar.ts       # Sidebar state management
│   │   │   ├── theme.ts         # Theme service (light/dark/system)
│   │   │   ├── localstorage.ts  # Local storage utilities
│   │   │   ├── consts.ts        # Template constants
│   │   │   ├── eventHub.ts      # Vue event bus
│   │   │   └── sharedBuefy.ts   # Shared Buefy notification/dialog
│   │   ├── components/          # Reusable Vue components
│   │   │   ├── Editor.vue       # CodeMirror markdown editor
│   │   │   ├── MarkdownPreview.vue # HTML preview for markdown content
│   │   │   ├── Header.vue       # Page header with nav & controls
│   │   │   ├── Calendar.vue     # Date picker calendar
│   │   │   ├── NoteCard.vue     # Note display card
│   │   │   ├── Tasks.vue        # Task list component
│   │   │   ├── Tags.vue         # Tag display component
│   │   │   ├── SimpleTask.vue   # Individual task item
│   │   │   ├── TaskItem.vue     # Task in list format
│   │   │   └── UnsavedForm.vue  # Unsaved changes dialog
│   │   └── views/               # Page-level Vue components (routed)
│   │       ├── Home.vue         # Main layout with sidebar
│   │       ├── Day.vue          # Daily note editor
│   │       ├── Note.vue         # Note detail page
│   │       ├── NewNote.vue      # Create new note
│   │       ├── Search.vue       # Search page
│   │       ├── Auth.vue         # Auth layout
│   │       ├── Login.vue        # Login form
│   │       ├── Signup.vue       # Registration form
│   │       └── Error pages      # 404, 401, error pages
│   ├── package.json             # Frontend dependencies & scripts
│   └── tsconfig.json            # TypeScript configuration
├── config/                       # Configuration directory
│   ├── app.db                   # SQLite database (default)
│   ├── .env                     # Environment variables (generated)
│   └── export.zip               # Export storage location
├── migrations/                   # Database migration files (Alembic)
├── config.py                    # Quart configuration
├── server.py                    # Quart app entry point (for uvicorn)
├── requirements.txt             # Python dependencies
├── run.sh                       # Production startup script
├── Dockerfile                   # Docker image definition
├── docker-compose.yml           # Docker compose configuration
└── README.md                    # Project documentation
```

## Core Architecture

### Data Flow

1. **Frontend (Vue.js)**

   - User interacts with Vue components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djedi/DailyNotes](https://github.com/djedi/DailyNotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
