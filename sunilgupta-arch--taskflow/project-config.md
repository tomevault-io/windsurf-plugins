---
trigger: always_on
description: TaskFlow is a Node.js/Express/EJS task management and work allocation platform with two completely isolated sides sharing one database and server:
---

# TaskFlow — Codebase Reference

## What This App Is

TaskFlow is a Node.js/Express/EJS task management and work allocation platform with two completely isolated sides sharing one database and server:

- **LOCAL side** — internal team/admin interface
- **CLIENT/Portal side** — external client-facing interface

---

## Tech Stack

- **Runtime**: Node.js, Express 4.18
- **Templates**: EJS 3.1 with `express-ejs-layouts`
- **Database**: MySQL2 3.6 (UTC storage, Eastern timezone in app)
- **Real-time**: Socket.IO 4.8 (dual namespace: `/` local, `/portal` client)
- **Auth**: JWT in `tf-token` cookie, bcryptjs, Google OAuth2
- **Email**: Nodemailer + Gmail SMTP (OAuth2 + app-password fallback) — `services/emailService.js`
- **Files**: Multer for uploads, Google Drive API integration
- **Scheduling**: node-cron (`utils/cronJobs.js`)
- **Migrations**: Auto-run on startup via `utils/auto-migrate.js` (71 SQL files in `migrations/`, latest is `072_weekly_roster_2026-07-03.sql`)

---

## Role System

Two distinct role families — middleware uses this to gate routes:

| Family | Roles | Access |
|--------|-------|--------|
| LOCAL | `LOCAL_ADMIN`, `LOCAL_MANAGER`, `LOCAL_USER` | `/` routes only |
| CLIENT | `CLIENT_ADMIN`, `CLIENT_MANAGER`, `CLIENT_USER`, `CLIENT_SALES` | `/portal` routes only |

`middleware/authenticate.js` blocks CLIENT roles from LOCAL routes and redirects them to `/portal`. `portal/middleware/portalOnly.js` enforces CLIENT roles on portal routes.

---

## Directory Structure

```
taskflow/
├── server.js                  # Entry point — Express, Socket.IO, migrations, cron
├── config/
│   ├── db.js                  # MySQL2 pool (UTC, dateStrings: true)
│   ├── socket.js              # Socket.IO init
│   ├── constants.js           # ROLES, PERMISSIONS, TASK_STATUS enums
│   └── multer.js              # File upload config
├── middleware/
│   ├── authenticate.js        # JWT auth + CLIENT role gating
│   ├── authorize.js           # Role/permission checks (requireRoles)
│   ├── spaJson.js             # X-SPA-Request: 1 → return JSON instead of HTML
│   ├── auditLog.js            # Action logging (CREATE, UPDATE, DELETE, etc.)
│   └── botDetect.js
├── routes/                    # LOCAL side routes
│   ├── index.js               # Admin hub, team, reports, queue, workspace
│   ├── tasks.js               # Task CRUD, board, sessions
│   ├── auth.js                # Login, logout, Google OAuth, profile
│   ├── chat.js                # Direct messaging
│   ├── drive.js               # Google Drive
│   └── help.js
├── controllers/               # LOCAL side business logic (~20 controllers)
│   ├── adminHubController.js  # New admin UI pages (1400+ lines)
│   ├── taskController.js      # Task board, sessions, completion (1200+ lines)
│   ├── reportController.js    # Analytics & reports (1100+ lines)
│   ├── devWorkspaceController.js  # Dev workspace (added May 2026)
│   ├── clientQueueController.js   # Client request queue
│   ├── authController.js
│   ├── chatController.js
│   └── ... (userController, leaveController, groupChannelController, etc.)
├── models/                    # LOCAL side DB abstraction (14 models)
│   ├── User.js, Task.js, TaskCompletion.js
│   ├── ClientRequest.js       # Client queue (large — 24K lines)
│   ├── DevProject.js          # Dev workspace projects
│   ├── Chat.js, GroupChannel.js, BridgeChat.js
│   ├── CompOff.js, LeaveRequest.js, Note.js, Reward.js
│   ├── ShiftHistory.js, Notification.js, Roster.js  # Roster.js: weekly weekoff planning (added Jul 2026)
├── services/                  # Shared business logic
│   ├── emailService.js        # Gmail SMTP, 5 email templates, OAuth2
│   ├── taskService.js         # Task helpers
│   ├── googleDriveService.js  # Drive API wrapper
│   ├── authService.js, backupService.js, dashboardService.js, linkUnfurl.js
├── utils/
│   ├── auto-migrate.js        # Runs pending SQL migrations on startup
│   ├── cronJobs.js            # Scheduled tasks
│   ├── timezone.js            # Eastern timezone helpers (DB is UTC)
│   ├── response.js            # ApiResponse.success/error/paginated
│   └── logger.js              # Winston logging
├── views/                     # LOCAL side EJS templates
│   ├── layouts/main.ejs       # Classic UI shell (Bootstrap 5)
│   ├── admin/                 # NEW Admin Hub (dark theme)
│   │   ├── layout.ejs         # Admin hub shell
│   │   ├── dashboard.ejs, work.ejs, queue.ejs, chat.ejs
│   │   ├── workspace.ejs      # Dev workspace (added May 2026)
│   │   ├── users.ejs, attendance.ejs, leaves.ejs, comp-off.ejs, roster.ejs
│   │   ├── my-tasks.ejs, my-attendance.ejs, my-progress.ejs
│   │   ├── taskboard.ejs, all-tasks.ejs, channel.ejs
│   │   └── infoboard.ejs, notes.ejs, drive.ejs, reports.ejs, ...
│   └── classic/               # Classic Bootstrap pages (tasks, users, reports, chat, etc.)
├── portal/                    # CLIENT side — completely separate stack
│   ├── routes/portal.js       # All /portal/* routes
│   ├── controllers/           # 6 portal controllers
│   │   ├── taskController.js, chatController.js, userController.js
│   │   ├── clientRequestController.js, urgentController.js, teamStatusController.js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sunilgupta-arch/taskflow](https://github.com/sunilgupta-arch/taskflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
