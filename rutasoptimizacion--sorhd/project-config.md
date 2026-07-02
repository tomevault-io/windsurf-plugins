---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**FlamenGO!** (Sistema de Optimización de Rutas para Hospitalización Domiciliaria) is a route optimization platform for home hospitalization services. The system helps clinical teams optimize daily visit routes considering personnel skills, vehicle capacity, time windows, and geographic constraints.

**Current Status**: This repository contains **planning and documentation only**. No implementation code exists yet. All development work should follow the specifications in this repository.

## Documentation Structure

### Core Documents

1. **specs/requirements.md** - Functional and non-functional requirements in EARS format
   - Read this FIRST to understand what the system must do
   - Contains requirements traceability matrix
   - Defines all user roles, entities, and constraints

2. **specs/design.md** - Comprehensive system design document
   - System architecture (3-tier: Backend, Web Admin, Mobile App)
   - Data model and database schema (PostgreSQL + PostGIS)
   - API specifications (RESTful endpoints)
   - Technology stack for each component
   - Security design (JWT authentication, RBAC)
   - Deployment architecture
   - Optimization algorithm design (OR-Tools VRP solver)

3. **CHECKLIST.md** - Detailed implementation checklist with 15 phases
   - Phase 0-1: Project setup and backend foundation
   - Phase 2-6: Backend modules (resources, optimization, tracking, notifications)
   - Phase 7-10: Admin panel (React.js)
   - Phase 11-13: Mobile app (React Native)
   - Phase 14-15: Integration, testing, and deployment
   - Each phase includes tasks, acceptance criteria, and dependencies

4. **documents/requerimientos_funcionales.md** - Original functional requirements (Spanish)

## System Architecture (High-Level)

### Three Main Components

1. **Backend** (Python + FastAPI)
   - PostgreSQL + PostGIS for geospatial data
   - OR-Tools for route optimization (VRP with time windows)
   - WebSocket for real-time GPS tracking
   - Push notifications (FCM/APNS) and SMS fallback

2. **Admin Panel** (React.js + TypeScript)
   - Resource management (personnel, vehicles, patients, cases)
   - Route planning interface with map visualization
   - Live monitoring dashboard (real-time vehicle tracking)

3. **Mobile App** (React Native)
   - Clinical Team profile: view routes, update visit status, GPS tracking
   - Patient profile: view visit status, track approaching team (Uber-style)

## Key Technical Decisions

### Backend
- **Framework**: FastAPI (async, type hints, auto-generated OpenAPI docs)
- **ORM**: SQLAlchemy with GeoAlchemy2 for PostGIS integration
- **Optimization**: Google OR-Tools (primary), custom heuristic (fallback)
- **Authentication**: JWT tokens with role-based access control (Admin, Clinical Team, Patient)
- **Real-time**: WebSocket for GPS location updates (30-second intervals)

### Database
- **PostgreSQL 15+** with **PostGIS** extension for geospatial queries
- Location stored as `GEOGRAPHY(POINT, 4326)` (WGS 84)
- Audit logging for all mutations
- Distance matrix caching (optional, 24-hour TTL)

### Admin Panel
- React 18 + TypeScript + Vite
- State: Redux Toolkit (global) + React Query (server state)
- UI: Material-UI
- Maps: Leaflet or Google Maps
- Real-time updates via WebSocket

### Mobile App
- React Native (iOS + Android)
- Redux Toolkit + AsyncStorage
- Background GPS tracking with minimal battery impact
- Offline support with sync queue
- Push notifications via Firebase (Android) and APNS (iOS)

## Route Optimization Algorithm

The core optimization problem is a **Vehicle Routing Problem with Time Windows and Skills (VRP-TWS)**.

**Constraints**:
- Personnel skill requirements must match case needs
- Vehicle capacity limits
- Time window constraints (e.g., "AM only", "10:00-12:00")
- Working hours (default 8:00-17:00)

**Objectives**:
1. Minimize total travel distance
2. Minimize total travel time
3. Balance workload across vehicles
4. Maximize on-time arrivals

**Implementation**:
- Primary: OR-Tools VRP solver (powerful, handles complex constraints)
- Fallback: Nearest neighbor + 2-opt local search (faster, approximate)

## Database Schema Highlights

**Key Tables**:
- `users` - Authentication (username, password_hash, role)
- `personnel` - Clinical staff (name, skills, start_location, work_hours)
- `vehicles` - Vehicles (identifier, capacity, base_location, resources)
- `patients` - Patient records (name, location)
- `cases` - Visit requests (patient_id, care_type, location, time_window, priority)
- `routes` - Optimized routes (vehicle_id, date, status, total_distance)
- `visits` - Individual visits in routes (route_id, case_id, sequence, times, status)
- `location_logs` - GPS tracking (vehicle_id, location, timestamp)
- `notifications` - Push/SMS notifications (user_id, type, message, delivery_status)

**Relationships**:
- Many-to-many: Personnel ↔ Skills, Routes ↔ Personnel
- One-to-many: Route → Visits, Vehicle → Routes, Patient → Cases

## API Structure

All endpoints under `/api/v1/`:

- `/auth/*` - Login, logout, token refresh
- `/personnel/*` - Personnel CRUD
- `/vehicles/*` - Vehicle CRUD

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rutasoptimizacion/sorhd](https://github.com/rutasoptimizacion/sorhd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
