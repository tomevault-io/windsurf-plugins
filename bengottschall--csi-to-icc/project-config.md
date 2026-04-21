---
trigger: always_on
description: This is a web application that helps civil engineers and construction professionals map CSI (Construction Specifications Institute) MasterFormat codes to ICC (International Code Council) building code sections.
---

# CSI to ICC Code Mapping - Project Context

## Project Overview

This is a web application that helps civil engineers and construction professionals map CSI (Construction Specifications Institute) MasterFormat codes to ICC (International Code Council) building code sections.

**Problem**: Converting CSI specification codes to relevant ICC compliance codes is a manual, time-consuming process requiring extensive cross-referencing.

**Solution**: A web application where users can:
- Enter a CSI code
- Filter by state and year
- Get relevant ICC code sections with direct links to official documentation

## Tech Stack

### Backend
- **Language**: Python 3.13+
- **Framework**: FastAPI (REST API)
- **Database**: PostgreSQL
- **ORM**: SQLAlchemy 2.0
- **Migrations**: Alembic
- **Validation**: Pydantic v2

### Frontend
- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript
- **UI Library**: React 19
- **Styling**: Tailwind CSS 3.4.1
- **HTTP Client**: fetch API

## Project Structure

```
csi-to-icc/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI application entry point
│   │   ├── database.py          # Database connection & session management
│   │   ├── models.py            # SQLAlchemy ORM models (5 tables)
│   │   ├── schemas.py           # Pydantic schemas for API validation
│   │   ├── crud.py              # Database CRUD operations
│   │   └── routers/
│   │       ├── __init__.py
│   │       └── codes.py         # API endpoints (21 endpoints)
│   ├── scripts/
│   │   ├── ipc_scraper_all.py   # Selenium-based ICC code scraper
│   │   ├── populate_icc_data.py # Database population from JSON
│   │   ├── populate_all_chapters.py  # Batch chapter processor
│   │   ├── extracted_data/
│   │   │   └── ipc_2018/        # 13 chapters of IPC 2018 (JSON)
│   │   ├── icc_credentials.env  # ICC login credentials (not in git)
│   │   └── README.md            # Scraper setup & usage guide
│   ├── alembic/                 # Database migrations
│   │   └── versions/
│   │       └── bc374c7d93a2_initial_schema.py
│   ├── venv/                    # Python virtual environment (not in git)
│   ├── requirements.txt         # Python dependencies
│   ├── add_sample_data.py       # Test data population script
│   ├── .env.example             # Environment variables template
│   └── .env                     # Actual environment variables (not in git)
├── frontend/
│   ├── app/
│   │   ├── page.tsx             # Main search interface (288 lines)
│   │   ├── layout.tsx           # Root layout
│   │   └── globals.css          # Tailwind configuration
│   ├── package.json             # Next.js 15, React 19, Tailwind 3.4
│   └── tailwind.config.ts       # Custom engineering-themed colors
├── docker-compose.yml           # Production Docker setup
├── docker-compose.dev.yml       # Development Docker setup
├── Makefile                     # Development commands
├── DOCKER.md                    # Docker deployment guide
├── .gitignore
├── CLAUDE.md                    # This file
└── README.md                    # User-facing documentation
```

## Frontend Implementation

### Search Interface (`app/page.tsx`)
**Features**:
- 3-field search form:
  - CSI Code input (required, e.g., "03 30 00")
  - State filter (optional, 2-letter format like "CO", "CA")
  - Year dropdown (optional, 2024/2021/2018)
- Real-time validation and error handling
- Loading states with spinner animation
- Empty state with example code suggestion

**Results Display**:
- CSI code information card (code, division, title, description)
- List of related ICC code sections
- Each section shows:
  - Document code and year (e.g., "IPC 2018")
  - Chapter number
  - Section number and title
  - Description text
  - Direct link to codes.iccsafe.org

**Design**:
- Custom Tailwind color palette:
  - Primary: Sky blue (#0ea5e9) - engineering/blueprint inspired
  - Accent: Construction orange (#f97316)
  - Neutral: Slate grays for professional look
- Responsive layout (mobile-first, tablet, desktop)
- Card-based design with shadows and borders
- Custom scrollbar styling
- Hover states and interactive elements

**API Integration**:
- Connects to backend at `http://localhost:8000`
- Uses fetch API for POST requests to `/api/search`
- Error handling for network failures and API errors
- Type-safe TypeScript interfaces for API responses

## Database Schema

### Tables

1. **csi_codes** - CSI MasterFormat codes
   - `id` (PK)
   - `code` (unique, e.g., "03 30 00")
   - `division` (integer)
   - `title` (e.g., "Cast-in-Place Concrete")
   - `description`

2. **icc_documents** - ICC code documents (IBC, IRC, etc.)
   - `id` (PK)
   - `code` (e.g., "IBC", "IRC")
   - `year` (e.g., 2021, 2024)
   - `title`
   - `state` (nullable, for state-specific codes)
   - `base_url` (link to codes.iccsafe.org)

3. **icc_sections** - Specific sections within ICC documents
   - `id` (PK)
   - `document_id` (FK to icc_documents)
   - `section_number` (e.g., "1905.2")
   - `title`
   - `chapter`
   - `url` (direct link to section)
   - `description`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BenGottschall) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
