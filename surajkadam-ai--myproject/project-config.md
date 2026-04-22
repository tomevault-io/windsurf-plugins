---
trigger: always_on
description: This is a full-stack Applicant Tracking System (ATS) built with React frontend and Node.js/Express backend, using MySQL database for data persistence. The system is designed for managing job requisitions, candidates, applications, interviews, and complete hiring workflows.
---

# CLAUDE.md - ATS Applicant Tracking System

## Project Overview
This is a full-stack Applicant Tracking System (ATS) built with React frontend and Node.js/Express backend, using MySQL database for data persistence. The system is designed for managing job requisitions, candidates, applications, interviews, and complete hiring workflows.

## Project Structure
```
Hkthon_ATS_latest_V5.24/
├── backend/                 # Node.js/Express API server
│   ├── src/
│   │   ├── config/         # MySQL database and Swagger configuration
│   │   ├── controllers/    # Request handlers for routes
│   │   ├── middleware/     # Auth, validation, error handling
│   │   ├── models/         # Data models (Application, Candidate, Interview, etc.)
│   │   ├── routes/         # API route definitions
│   │   ├── utils/          # Utility functions
│   │   └── server.js       # Main Express server
│   ├── scripts/            # Database seeding scripts
│   └── package.json        # Backend dependencies
├── frontend/               # React application
│   ├── src/
│   │   ├── components/     # Reusable UI components (shadcn/ui)
│   │   ├── pages/          # Page components
│   │   ├── services/       # API service layer (axios)
│   │   ├── store/          # Redux store and slices
│   │   ├── contexts/       # React contexts
│   │   ├── hooks/          # Custom React hooks
│   │   ├── lib/            # Utility functions
│   │   ├── utils/          # Helper utilities
│   │   ├── App.jsx         # Main app component
│   │   └── main.jsx        # App entry point
│   ├── public/             # Static assets
│   └── package.json        # Frontend dependencies
└── package.json            # Root package scripts
```

## Technology Stack

### Backend
- **Framework**: Express.js 4.19.2
- **Database**: MySQL 9.4 with mysql2 driver (3.15.1)
- **Authentication**: JWT (jsonwebtoken 9.0.2) with bcryptjs 2.4.3
- **Validation**: express-validator 7.1.0, joi 17.13.3
- **Security**: helmet, cors
- **File Upload**: multer 2.0.2
- **Documentation**: Swagger UI Express + swagger-jsdoc
- **Development**: nodemon 3.1.0

### Frontend
- **Framework**: React 18.3.1
- **Routing**: React Router DOM 6.28.0
- **State Management**: Redux Toolkit 2.2.6 (@reduxjs/toolkit)
- **HTTP Client**: Axios 1.7.9
- **UI Components**: shadcn/ui built on Radix UI primitives
  - @radix-ui/react-dialog, select, tabs, toast, etc.
- **Styling**: Tailwind CSS 3.4.14 with tailwindcss-animate
- **Forms**: React Hook Form 7.52.2 with @hookform/resolvers
- **Validation**: Zod 3.23.8
- **Date Handling**: date-fns 3.6.0
- **Icons**: Lucide React 0.427.0
- **Charts**: Recharts 3.2.1
- **Drag & Drop**: react-dnd 16.0.1 with html5-backend
- **Tables**: @tanstack/react-table 8.21.3
- **Build Tool**: Vite 5.3.4
- **Utilities**: clsx, class-variance-authority, tailwind-merge

## Database Schema

### MySQL Database: `ats_db`

**Main Tables:**
1. **users** - System users (admin, hr_manager, recruiter, interviewer, hiring_manager)
2. **jobRequisitions** - Job postings and requisitions
3. **candidates** - Candidate profiles with skills and experience
4. **applications** - Job applications linking candidates to jobs
5. **interviews** - Interview scheduling and feedback
6. **auditLogs** - System audit trail

**Key Fields:**
- `candidate_code` - Human-readable candidate identifier (CAND-XXX)
- `application_code` - Human-readable application identifier (APPL-XXX)
- `scheduledDate` - DATETIME field for interview scheduling
- `interviewerId` - Single interviewer assignment (not array)

## Development Commands

### Installation
```bash
# Install all dependencies (backend + frontend)
npm run install:all

# Install backend only
npm run install:backend

# Install frontend only
npm run install:frontend
```

### Development
```bash
# Run both frontend and backend concurrently
npm run dev

# Run backend only (port 5000)
npm run dev:backend

# Run frontend only (port 3000)
npm run dev:frontend
```

### Production
```bash
# Build frontend for production
npm run build

# Start backend in production mode
npm start
```

### Database Management
```bash
# Seed job requisitions
npm run seed:job-requisitions

# Clear job requisitions
npm run clear:job-requisitions
```

### Code Quality
```bash
# Frontend linting
cd frontend && npm run lint
cd frontend && npm run lint:fix
```

### Port Management
```bash
# Kill ports if needed
npx kill-port 3000 5000

# Quick start script
start-dev.bat
```

## Key Features

### Core Functionality
- **User Management**: Role-based authentication and authorization
- **Job Requisitions**: Create, manage, and track job postings
- **Candidate Management**:
  - Profile management with candidate codes (CAND-XXX)
  - Skills tracking (primary and other skills)
  - Resume/document upload
  - Bulk candidate import via CSV
- **Application Workflow**:
  - Application tracking with codes (APPL-XXX)
  - Status management (applied, screening, interviewed, assessment, offer, hired, rejected, withdrawn)
  - Stage tracking throughout hiring pipeline
  - Candidate-to-job mapping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SurajKadam-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
