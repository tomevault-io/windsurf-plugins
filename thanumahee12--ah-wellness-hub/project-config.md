---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AH Wellness Hub Blood Lab Manager is a complete Point of Sale (POS) system for blood testing laboratories. It provides comprehensive management of patients, blood tests, checkups/billing, medicines, user management with activity tracking, and generates professional PDF invoices and prescriptions. The application features comprehensive role-based access control (RBAC) with superadmin, admin, maintainer, editor, and user roles.

## Development Commands

- **Start dev server**: `npm run dev` - Starts Vite dev server with HMR at http://localhost:5173
- **Build**: `npm run build` - Production build to `dist/` directory
- **Lint**: `npm run lint` - Run ESLint on all files
- **Preview build**: `npm run preview` - Preview production build locally

## Tech Stack

- **Framework**: React 19.1.1
- **Build Tool**: Vite 7.1.7
- **State Management**: Redux Toolkit (@reduxjs/toolkit 2.9.0)
- **Routing**: React Router DOM 7.9.4
- **UI Framework**: React Bootstrap 2.10.10 + Bootstrap 5.3.8
- **Icons**: React Icons 5.5.0
- **PDF Generation**: jsPDF 3.0.3 + html2canvas 1.4.1
- **Charts**: Recharts 2.15.0
- **Backend**: Firebase (Authentication, Firestore, Hosting)
- **Language**: JavaScript (JSX)

## Project Structure

### Core Files
- `src/main.jsx` - Entry point with Redux Provider, Router, and Bootstrap CSS
- `src/App.jsx` - Main app with routing and protected routes
- `index.html` - HTML shell

### State Management (Redux)
- `src/store/store.js` - Redux store configuration
- `src/store/authSlice.js` - Authentication state with Firebase integration
- `src/store/testsSlice.js` - Blood tests management with Firestore sync
- `src/store/patientsSlice.js` - Patients management with Firestore sync
- `src/store/checkupsSlice.js` - Checkups/billing management with Firestore sync
- `src/store/usersSlice.js` - Users management with Firestore sync
- `src/store/medicinesSlice.js` - Medicines management with Firestore sync
- `src/store/settingsSlice.js` - Dynamic settings (forms, tables, pages, permissions) with Firestore sync

### Components
- `src/components/Navbar.jsx` - Top navigation bar with user info and logout
- `src/components/Sidebar.jsx` - Side navigation with role-based menu items
- `src/components/ProtectedRoute.jsx` - Route protection with role-based access control
- `src/components/crud/` - Reusable CRUD components (CRUDTable, CRUDModal, EnhancedCRUDTable)
- `src/components/ui/` - UI components (PageHeader, FormField, RichTextEditor)

### Pages
- `src/pages/Home.jsx` - Landing page with features showcase (content driven by settings)
- `src/pages/Login.jsx` - Firebase authentication page
- `src/pages/Dashboard.jsx` - Analytics dashboard with charts, statistics, and time-range filters
- `src/pages/Patients.jsx` - Patient CRUD with enhanced table
- `src/pages/PatientDetail.jsx` - Individual patient details and history
- `src/pages/Checkups.jsx` - Checkup/billing with multi-step form
- `src/pages/CheckupDetail.jsx` - Detailed checkup view with invoice and prescription PDF generation
- `src/pages/Tests.jsx` - Blood tests CRUD (role-restricted)
- `src/pages/Medicines.jsx` - Medicine inventory management (role-restricted)
- `src/pages/UserManagement.jsx` - Consolidated user management with tabs:
  - `src/pages/tabs/UsersTab.jsx` - User CRUD operations
  - `src/pages/tabs/UserActivityTab.jsx` - Activity tracking and analytics (superadmin only)
  - `src/pages/tabs/UserRequestsTab.jsx` - User role change requests (superadmin only)
- `src/pages/Settings.jsx` - Settings page with dynamic tabs
  - `src/pages/tabs/TablesSettingsTab.jsx` - Table column config (visibility, roles, searchable)
  - `src/pages/tabs/PagesSettingsTab.jsx` - Page access and sidebar config
  - `src/pages/tabs/PublicPageTab.jsx` - Home page content editor (hero, blogs, CTA)
- `src/pages/ColumnSettingDetail.jsx` - Column detail editor (label, visible, roles, searchable, field type, etc.)
- `src/pages/AdminSetup.jsx` - Initial admin setup page

### Services
- `src/services/authService.js` - Firebase Authentication service
- `src/services/firestoreService.js` - Firestore database operations
- `src/services/activityService.js` - User activity tracking
- `src/services/roleRequestService.js` - Role change request management

### Hooks
- `src/hooks/useCRUD.js` - Reusable CRUD operations hook
- `src/hooks/usePermission.js` - Permission checking hook
- `src/hooks/useSettings.js` - Settings access hook (filter fields/columns by visibility, role, searchable; page access checks; permission checks)

### Constants
- `src/constants/defaultSettings.js` - Default settings (forms, tables, pages, permissions); single source of truth merged with Firestore
- `src/constants/roles.js` - Role hierarchy and permission helpers

### Context
- `src/context/NotificationContext.jsx` - Toast notifications provider

## Application Features

### Authentication & Authorization
- Firebase Authentication integration
- Comprehensive role-based access control (RBAC)
- Fine-grained permissions system (read, create, edit, delete per resource)
- User activity tracking for audit trails
- Role change request workflow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThanuMahee12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
