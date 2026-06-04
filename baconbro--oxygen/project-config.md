---
trigger: always_on
description: Oxygen is an open-source work management platform and Jira alternative built with React and Firebase. It provides issue tracking, agile boards (Scrum/Kanban), project planning with epics and stories, sprint management, OKR/goals tracking, and work package management.
---

# CLAUDE.md - AI Assistant Guide for Oxygen

## Project Overview

Oxygen is an open-source work management platform and Jira alternative built with React and Firebase. It provides issue tracking, agile boards (Scrum/Kanban), project planning with epics and stories, sprint management, OKR/goals tracking, and work package management.

**Live Demo:** https://oxgn.io

## Technology Stack

- **Frontend:** React 18, TypeScript (mixed with JavaScript)
- **State Management:** React Query, React Context, Redux Toolkit
- **Backend:** Firebase (Firestore, Authentication, Hosting)
- **Styling:** SCSS with Bootstrap 5, CSS variables for theming
- **Build Tool:** Create React App (react-scripts)
- **UI Libraries:** React Bootstrap, FullCalendar, Gantt charts, Quill editor, React Beautiful DnD

## Quick Commands

```bash
# Install dependencies
npm install

# Start development server
npm start

# Build for production
npm run build

# Run tests
npm test

# Format code with Prettier
npm run format

# Check formatting
npm run lint

# Start Firebase emulators (for local development)
firebase emulators:start

# Deploy to Firebase
firebase deploy --only hosting
```

## Project Structure

```
/home/user/Oxygen/
├── src/
│   ├── App.tsx              # Root app component with providers
│   ├── index.tsx            # Entry point
│   ├── components/          # Shared/reusable components
│   │   ├── common/          # Generic UI components (Avatar, Button, Modal, Form, etc.)
│   │   ├── partials/        # Partial components
│   │   └── ErrorBoundary.jsx
│   ├── contexts/            # React contexts
│   │   ├── AuthContext.jsx  # Authentication context
│   │   └── WorkspaceProvider.jsx  # Workspace state context
│   ├── hooks/               # Custom React hooks
│   │   ├── api/             # API-related hooks
│   │   └── *.js             # Utility hooks
│   ├── i18n/                # Internationalization
│   ├── layout/              # Layout components (MasterLayout, sidebars)
│   ├── modules/             # Feature modules (main business logic)
│   │   ├── auth/            # Authentication (login, register)
│   │   ├── admin/           # Admin panel
│   │   ├── Goals/           # OKR/Goals management
│   │   ├── home/            # Dashboard home
│   │   ├── IssueDetails/    # Issue viewing/editing (23 subcomponents)
│   │   ├── Org/             # Organization management
│   │   ├── Workspace/       # Project/workspace features (board, backlog, sprints)
│   │   ├── User/            # User profile management
│   │   ├── accounts/        # Account settings
│   │   ├── onboarding/      # New user onboarding
│   │   └── errors/          # Error pages
│   ├── pages/               # Page-level components
│   ├── redux/               # Redux store configuration
│   ├── routing/             # React Router configuration
│   │   ├── AppRoutes.tsx    # Public routes
│   │   └── PrivateRoutes.tsx # Authenticated routes
│   ├── services/            # Firebase/API service layer
│   │   ├── firestore.js     # Core Firebase operations
│   │   ├── itemServices.js  # Issue/task CRUD
│   │   ├── sprintServices.js # Sprint management
│   │   ├── okrServices.js   # OKR/Goals services
│   │   ├── workspaceServices.js # Workspace CRUD
│   │   ├── workPackageServices.js # Work packages
│   │   └── userServices.js  # User management
│   ├── styles/              # SCSS styles
│   │   ├── core/            # Core styling (variables, mixins, components)
│   │   └── layout/          # Layout-specific styles
│   └── utils/               # Utility functions
├── public/                  # Static assets
├── docs/                    # Documentation
└── package.json
```

## Key Modules

### IssueDetails (`/src/modules/IssueDetails/`)
The most complex module with 23+ subcomponents for viewing/editing issues:
- Title, Description, Status, Priority selectors
- Checklist management
- Sub-issues (SubsComponent)
- Task dependencies
- Comments, attachments

### Workspace (`/src/modules/Workspace/`)
Project workspace with multiple views:
- Board (Kanban with drag-and-drop)
- Backlog
- Sprints
- Timeline/Gantt
- Calendar
- List view

### Goals (`/src/modules/Goals/`)
OKR and objectives tracking with progress measurement.

## Firebase/Firestore Data Model

```
organisation/
└── {orgId}/
    └── spaces/
        └── {spaceId}/
            ├── config (issueStatus, issueType, workspaceConfig)
            ├── items/ (issues/tasks)
            ├── sprints/
            │   └── tickets/ (sprint-task associations)
            ├── workpackages/
            ├── goals/
            ├── userviews/
            └── issueHistory/

users/
└── {uid}/ (user profiles)
```

## Code Conventions

### File Naming
- React components: PascalCase (e.g., `IssueDetails.jsx`, `WorkspaceProvider.jsx`)
- Services: camelCase with "Services" suffix (e.g., `itemServices.js`)
- Hooks: camelCase with "use" prefix (e.g., `useQueryString.js`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baconbro/Oxygen](https://github.com/baconbro/Oxygen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
