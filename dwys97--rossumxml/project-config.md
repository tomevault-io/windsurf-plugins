---
trigger: always_on
description: **Production-ready XML mapping, transformation, and AI-powered invoice extraction system with enterprise security.**
---

# Project Overview: SCHEMABRIDGE - Enterprise XML Transformation Platform

**Production-ready XML mapping, transformation, and AI-powered invoice extraction system with enterprise security.**

## A. Current Development Setup and Tech Stack

### 🏗️ Architecture Overview
**Monorepo** with Frontend, Backend, ML Service, WebSocket Server, and Background Workers.

### **Frontend (Directory: `frontend/`)**
- **Stack:** React 19 + Vite 7
- **Dev Environment:** Port 5173, HMR enabled
- **API Proxy:** All `/api` requests → `http://localhost:3000`
- **Key Features:**
  - Visual XML mapping editor with drag-and-drop
  - Interactive schema tree viewer
  - AI mapping suggestions UI (75-90% accuracy)
  - Admin dashboard (user management, RBAC, security monitoring)
  - PDF invoice viewer with real-time extraction
  - Glassmorphic UI with auto-logout (10min idle)
- **Dependencies:** React Router, Socket.io-client, Chart.js, jsPDF, react-pdf, TanStack Table

### **Backend (Directory: `backend/`)**
- **Stack:** AWS SAM (Lambda + API Gateway) + Node.js 18
- **Dev Environment:** Port 3000 (SAM Local)
- **Database:** PostgreSQL 13 (Docker, port 5432)
- **Core Services:**
  - **XML Transformation** (`/api/transform`, `/api/webhook/transform`)
  - **Schema Parsing** (`/api/schema/parse`)
  - **AI Mapping** (`/api/ai/suggest-mapping`, `/api/ai/batch-suggest`)
  - **Invoice Extraction** (`/api/invoice/*` - OCR + LayoutLMv3 + Gemini)
  - **Rossum Integration** (`/api/webhook/rossum`)
  - **Admin APIs** (`/api/admin/*` - users, roles, security audit)
  - **Security** - RBAC, JWT auth, XML security validation, audit logging
- **Key Services:**
  - `xmlParser.service.js` - **PROTECTED** XML parsing & transformation logic
  - `aiMapping.service.js` - AI-powered field mapping (Gemini 2.0 Flash)
  - `invoiceExtraction.service.js` - ML-based invoice data extraction
  - `user.service.js` - User management with RBAC
  - `selfLearning.service.js` - Self-learning from user corrections
- **Dependencies:** Express, @xmldom/xmldom, pg (PostgreSQL), jsonwebtoken, bcrypt, @google/generative-ai, Bull (job queue), Socket.io, Redis, Sharp, Multer

### **ML Service (Directory: `services/`)**
- **Stack:** Python 3.10 + Flask
- **Architecture (Ultra-Lightweight <6GB):** 
  - **P1: OCR Service** - PaddleOCR + PP-Structure (~500MB)
  - **P2: Extractor Service** - GLiNER small model (~300MB)
  - **P3: API Gateway** - FastAPI + Label Studio HITL integration (~100MB)
- **Dev Ports:** 5002 (OCR), 5003 (Extractor), 8000 (Gateway)
- **Purpose:** CPU-only invoice extraction with spatial context augmentation
- **HITL:** Automatic routing to Label Studio when confidence < 0.90
- **Proven Architecture:** See `.github/extraction_arch.md`

### **WebSocket Server (File: `backend/socketServer.js`)**
- **Stack:** Socket.io + Node.js
- **Dev Environment:** Port 3001
- **Purpose:** Real-time updates for invoice extraction jobs
- **Features:** Job progress, extraction status, live error reporting

### **Background Workers (Directory: `backend/workers/`)**
- **Stack:** Bull (Redis-backed job queue)
- **Purpose:** Asynchronous invoice processing
- **Worker:** `extractionWorker.js` - Handles ML inference jobs

### **Database (PostgreSQL 13)**
- **Container:** Docker Compose, port 5432
- **Schema:** `backend/db/init.sql` + migrations (`backend/db/migrations/`)
- **Key Tables:**
  - `users`, `organizations`, `roles`, `permissions` (RBAC)
  - `mappings`, `schemas` (XML transformation)
  - `invoices`, `line_items`, `invoice_audit_log` (invoice extraction)
  - `security_audit_log` (security events)
  - `api_keys`, `webhooks` (API management)
- **Features:** Row-Level Security (RLS), audit logging, multi-tenancy

### **API Endpoints Overview**

#### Core Microservices (Ultra-Lightweight IDP)
- **P1 OCR Service:** `POST http://localhost:5002/process-document` - PaddleOCR + spatial context
- **P2 Extractor Service:** `POST http://localhost:5003/extract-customs-fields` - GLiNER NER (~300MB)
- **P3 API Gateway:** `POST http://localhost:8000/api/v1/invoice/upload` - HITL orchestration
- **Label Studio:** `http://localhost:8080` - Human-in-the-Loop corrections

#### Legacy Transformation & Mapping
- `POST /api/transform` - Synchronous XML transformation
- `POST /api/webhook/transform` - Async webhook transformation
- `POST /api/schema/parse` - Parse XML to tree structure
- `POST /api/ai/suggest-mapping` - AI field mapping (single)
- `POST /api/ai/batch-suggest` - AI field mapping (batch)

#### Invoice Extraction (AI/ML)
- `POST /api/invoice/upload` - Upload invoice (PDF/image)
- `GET /api/invoice/:id` - Get extraction results
- `POST /api/invoice/:id/correct` - Submit user corrections (self-learning)
- `GET /api/invoice/:id/audit` - Get audit trail
- `GET /api/analytics/accuracy` - ML accuracy metrics

#### Rossum Integration
- `POST /api/webhook/rossum` - Rossum webhook receiver
- Converts Rossum JSON → XML → Transformation → Destination webhook

#### Admin & Security
- `POST /api/auth/login` - JWT authentication
- `GET /api/admin/users` - User management (Admin only)
- `POST /api/admin/users` - Create user
- `PUT /api/admin/users/:id` - Update user/roles
- `DELETE /api/admin/users/:id` - Delete user

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dwys97/ROSSUMXML](https://github.com/Dwys97/ROSSUMXML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
