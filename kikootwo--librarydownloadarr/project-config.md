---
trigger: always_on
description: This document provides a comprehensive overview of the LibraryDownloadarr codebase to help Claude agents (or any AI assistant) get up to speed quickly and work effectively on this project.
---

# CLAUDE.md - Developer Reference for AI Agents

This document provides a comprehensive overview of the LibraryDownloadarr codebase to help Claude agents (or any AI assistant) get up to speed quickly and work effectively on this project.

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture & Tech Stack](#architecture--tech-stack)
3. [Key Directories & Files](#key-directories--files)
4. [Development Workflow](#development-workflow)
5. [Common Tasks](#common-tasks)
6. [Important Patterns & Conventions](#important-patterns--conventions)
7. [Mobile & PWA Considerations](#mobile--pwa-considerations)
8. [Testing & Building](#testing--building)
9. [Git Workflow](#git-workflow)
10. [Known Issues & Gotchas](#known-issues--gotchas)
11. [API Overview](#api-overview)
12. [Authentication Flow](#authentication-flow)

---

## Project Overview

**LibraryDownloadarr** is a modern web application that provides a user-friendly interface for downloading media from a Plex Media Server. It integrates with Plex's authentication system and respects user permissions.

### Key Features
- **Plex OAuth Authentication**: Users sign in with their Plex accounts
- **Library Browsing**: Display movies, TV shows, and music with posters and metadata
- **One-Click Downloads**: Download original media files directly
- **Permission Respect**: Honors Plex's user access controls
- **Progressive Web App (PWA)**: Installable on mobile devices
- **Responsive Design**: Works on desktop, mobile browser, and PWA
- **Download Management**: Real-time download progress with queue management
- **Admin Features**: Download history, logs, and settings

### Project Goals
- Provide a sleek, Overseerr-like interface for Plex downloads
- Respect Plex server permissions and user access
- Work seamlessly on mobile and desktop
- Maintain security through proper authentication

---

## Architecture & Tech Stack

### Backend
- **Runtime**: Node.js with Express
- **Language**: TypeScript
- **Database**: SQLite (via better-sqlite3)
- **Authentication**: Session-based with express-session
- **Plex Integration**: Direct Plex API calls (no official SDK)
- **Logging**: Winston for structured logging
- **File Serving**: Direct file streaming from Plex server paths

**Location**: `/backend`

### Frontend
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS with custom dark theme
- **State Management**: Zustand for auth, React Context for downloads
- **Routing**: React Router v6
- **API Client**: Axios
- **PWA**: Service Worker + Web App Manifest

**Location**: `/frontend`

### Deployment
- **Containerization**: Docker with docker-compose
- **Multi-stage Build**: Frontend built and served by backend
- **Volumes**: SQLite database and logs persisted
- **Network**: Backend serves both API and static frontend

---

## Key Directories & Files

### Backend Structure
```
backend/
├── src/
│   ├── index.ts                 # Entry point, Express setup
│   ├── config/
│   │   └── index.ts            # Configuration (ports, Plex metadata)
│   ├── db/
│   │   ├── index.ts            # SQLite database initialization
│   │   └── schema.sql          # Database schema
│   ├── middleware/
│   │   ├── auth.ts             # Authentication middleware
│   │   └── error.ts            # Error handling middleware
│   ├── routes/
│   │   ├── auth.ts             # Login, logout, Plex OAuth
│   │   ├── media.ts            # Media browsing and downloads
│   │   ├── libraries.ts        # Library listing
│   │   ├── settings.ts         # Admin settings (Plex URL/token)
│   │   ├── downloads.ts        # Download history
│   │   └── logs.ts             # System logs
│   ├── services/
│   │   └── plexService.ts      # Plex API integration
│   ├── utils/
│   │   └── logger.ts           # Winston logger setup
│   └── types/
│       └── index.ts            # TypeScript types
├── package.json
└── tsconfig.json
```

### Frontend Structure
```
frontend/
├── src/
│   ├── main.tsx                # Entry point, React setup
│   ├── App.tsx                 # Routes and protected route logic
│   ├── components/
│   │   ├── Header.tsx          # Top navigation with safe area insets
│   │   ├── Sidebar.tsx         # Left nav with library list
│   │   ├── DownloadManager.tsx # Floating download progress widget
│   │   └── MediaCard.tsx       # Reusable media poster card
│   ├── pages/
│   │   ├── Setup.tsx           # First-time admin setup
│   │   ├── Login.tsx           # Login with Plex OAuth
│   │   ├── Dashboard.tsx       # Home page with libraries
│   │   ├── LibraryView.tsx     # Grid view of media in library
│   │   ├── MediaDetail.tsx     # Detail page with download button
│   │   ├── SearchResults.tsx   # Search results page
│   │   ├── Settings.tsx        # Admin settings page
│   │   ├── DownloadHistory.tsx # Admin download history
│   │   └── Logs.tsx            # Admin logs page
│   ├── stores/
│   │   └── authStore.ts        # Zustand store for auth state
│   ├── contexts/
│   │   └── DownloadContext.tsx # React Context for download queue
│   ├── services/
│   │   └── api.ts              # Axios API client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kikootwo/LibraryDownloadarr](https://github.com/kikootwo/LibraryDownloadarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
