---
trigger: always_on
description: > **Purpose**: Rapid onboarding for AI agents and developers. Read this first before touching any code.
---

# OpenCodeHub — Agent Reference Guide

> **Purpose**: Rapid onboarding for AI agents and developers. Read this first before touching any code.

---

## 1. What Is This?

**OpenCodeHub** is a self-hosted, open-source alternative to GitHub/GitLab. It is a **modular monolith**: one main web app + optional background workers, with pluggable persistence and integrated Git protocol handling.

**Key differentiators**:
- Stack-first PR workflows (Graphite-style stacked PRs)
- Merge queue with speculative builds and priority lanes
- GitHub Actions-compatible CI/CD pipeline engine + Docker-based runner
- Pluggable storage (`local` filesystem or `s3` — any S3-compatible object store: AWS S3, MinIO, Cloudflare R2, Garage, SeaweedFS, Ceph RGW, Wasabi, Backblaze B2, etc.)
- Multi-database support (PostgreSQL, SQLite, Turso/LibSQL)
- Built-in AI code review, automations, webhooks

---

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENTS                               │
│  Browser  │  Git CLI (HTTP/SSH)  │  OpenCodeHub CLI (och)   │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    OPENCODEHUB PLATFORM                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Web UI     │  │  REST API   │  │  GraphQL Endpoint   │  │
│  │  (Astro+React)│  │  (140+ routes)│  │  (src/pages/api/)   │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Git Server │  │  SSH Server │  │  Pipeline Runner    │  │
│  │  (HTTP RPC) │  │  (ssh2)     │  │  (Docker executor)  │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│              PERSISTENCE & INFRASTRUCTURE                    │
│  PostgreSQL/SQLite/Turso  │  Redis  │  Pluggable Storage    │
└─────────────────────────────────────────────────────────────┘
```

### Runtime Processes
| Process | Command | Purpose |
|---------|---------|---------|
| Main App | `npm run dev` / `astro dev` | Web UI + API + Git HTTP |
| SSH Git | `npm run git:start` | SSH git push/pull server |
| Worker | `npm run worker:start` | Background jobs (queues, webhooks) |
| Runner | `npm run runner:start` | CI/CD pipeline execution |

---

## 3. Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Astro 4.x (SSR mode, `@astrojs/node` standalone adapter) |
| UI | React 18 + Tailwind CSS + Radix UI primitives |
| Database ORM | Drizzle ORM |
| DB Drivers | PostgreSQL (pg), SQLite (better-sqlite3), LibSQL/Turso |
| Auth | JWT (jose) + bcryptjs + TOTP (otplib) + OAuth (arctic) |
| Git | Native `git` CLI + `simple-git` + `isomorphic-git` + `nodegit` |
| SSH | `ssh2` library |
| CI/CD | Dockerode (Docker API) |
| Storage | Abstract adapter pattern; 2 backends (`local`, `s3`-compatible) |
| Realtime | Custom (see `src/lib/realtime.ts`) |
| Queue | BullMQ + Redis |
| CLI | Commander.js + Inquirer + Chalk + `simple-git` |

---

## 4. Directory Structure

```
/home/swadhin/owngit/OpenCodeHub/
├── src/                          # MAIN APPLICATION
│   ├── pages/                    # Astro file-based routing
│   │   ├── api/                  # REST API routes (140+ files)
│   │   │   ├── auth/             # Login, register, OAuth, 2FA
│   │   │   ├── repos/            # Repository CRUD, git endpoints
│   │   │   ├── actions/          # CI/CD pipeline APIs
│   │   │   ├── admin/            # Admin endpoints
│   │   │   ├── user/             # User settings, PATs
│   │   │   ├── stacks/           # Stacked PR APIs
│   │   │   ├── graphql.ts        # GraphQL endpoint
│   │   │   └── openapi.json.ts   # OpenAPI spec endpoint
│   │   ├── [owner]/[repo]/       # Repository pages (issues, PRs, wiki, etc.)
│   │   ├── admin/                # Admin dashboard pages
│   │   ├── git/                  # Git HTTP protocol endpoints
│   │   └── ...                   # Other pages (login, settings, etc.)
│   ├── lib/                      # CORE BUSINESS LOGIC (120+ modules)
│   │   ├── auth.ts               # JWT, sessions, password hashing, 2FA
│   │   ├── git-server.ts         # Git HTTP RPC (upload-pack/receive-pack)
│   │   ├── ssh.ts                # SSH git server implementation
│   │   ├── storage.ts            # Pluggable storage adapters
│   │   ├── pipeline.ts           # CI/CD workflow engine
│   │   ├── stacks.ts             # Stacked PR core logic
│   │   ├── merge-queue.ts        # Merge queue with speculative builds
│   │   ├── pull-requests.ts      # PR CRUD and merge logic
│   │   ├── permissions.ts        # RBAC and repo access control
│   │   ├── webhooks.ts           # Outbound webhook delivery
│   │   ├── automations.ts        # Workflow automation rules
│   │   ├── ai-review.ts          # AI-powered code review
│   │   ├── validation.ts         # Input validation (Zod)
│   │   └── ...                   # Many more specialized modules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swadhinbiswas/OpencodeHub](https://github.com/swadhinbiswas/OpencodeHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
