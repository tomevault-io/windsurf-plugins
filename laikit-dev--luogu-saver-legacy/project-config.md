---
trigger: always_on
description: **Luogu Saver** is a web application for archiving and preserving content from Luogu (a Chinese online judge platform). It's a Node.js-based Express web application with server-side rendering using Nunjucks templates and SemanticUI for the frontend.
---

# Luogu Saver - Copilot Coding Instructions

## Project Overview

**Luogu Saver** is a web application for archiving and preserving content from Luogu (a Chinese online judge platform). It's a Node.js-based Express web application with server-side rendering using Nunjucks templates and SemanticUI for the frontend.

**Project Type**: Full-stack web application (SSR)  
**Lines of Code**: ~5,300 lines of JavaScript  
**Languages**: JavaScript (ES Modules), Nunjucks templates, JSON  
**Runtime**: Node.js 22.18.0 (specified in README)  
**Package Manager**: npm (package-lock.json is in .gitignore, but npm install works)

### Key Technologies
- **Backend**: Express 5.x, TypeORM 0.3.x
- **Database**: MySQL/MariaDB (via mysql2), Redis (via ioredis)
- **Frontend**: Nunjucks templates, SemanticUI, jQuery, KaTeX (math rendering)
- **Task Scheduling**: node-schedule
- **Markdown**: markdown-it with plugins (attrs, container)
- **Real-time**: WebSocket (ws library)

## Directory Structure

```
/
├── .github/                  # GitHub workflows and issue templates
│   └── workflows/
│       └── autoAssign.yml    # Auto-assigns advertising issues
├── app.js                    # Main application entry point
├── package.json              # Project dependencies and scripts
├── migrate.js                # Database migration script
├── config.example.js         # Configuration template (copy to config.js)
├── ormconfig.example.json    # TypeORM config template (copy to ormconfig.json)
├── contentConfig.example.json     # App settings template (copy to contentConfig.json)
├── accounts.example.json     # Accounts config template (copy to accounts.json)
├── core/                     # Core utilities and helpers
│   ├── logger.js             # Logging with chalk (info, warn, error, debug)
│   ├── utils.js              # Utility functions (formatDate, hash, etc.)
│   ├── markdown.js           # Markdown rendering configuration
│   ├── redis.js              # Redis client manager
│   ├── request.js            # HTTP request utilities
│   ├── cache.js              # Caching logic
│   ├── pagination.js         # Pagination helpers
│   ├── response.js           # Response formatting
│   └── storage.js            # Storage utilities
├── entities/                 # TypeORM entity definitions (JSON format)
│   ├── index.js              # Entity loader
│   ├── article.entity.json   # Article entity schema
│   ├── user.entity.json      # User entity schema (basic info only)
│   ├── user_introduction.entity.json # User introduction data (separate table)
│   ├── paste.entity.json     # Paste entity schema
│   ├── problem.entity.json   # Problem entity schema
│   ├── task.entity.json      # Task entity schema
│   ├── judgement.entity.json # Judgement entity schema
│   └── [other entities]
├── models/                   # Business logic models
│   ├── common.js             # Common model utilities
│   ├── article.js            # Article model
│   ├── user.js               # User model
│   ├── user_introduction.js  # User introduction model
│   └── [other models]
├── services/                 # Service layer (14 services)
│   ├── article.service.js    # Article business logic
│   ├── user.service.js       # User business logic
│   ├── admin.service.js      # Admin operations
│   ├── problem.service.js    # Problem management
│   ├── statistic.service.js  # Statistics
│   └── [other services]
├── routes/                   # Express route definitions (14 route files)
│   ├── index.route.js        # Home page routes
│   ├── article.route.js      # Article routes
│   ├── admin.route.js        # Admin panel routes
│   ├── benben.route.js       # Benben (chat/feed) routes
│   ├── problem.route.js      # Problem routes
│   └── [other routes]
├── middleware/               # Express middleware
│   ├── auth.js               # Authentication middleware
│   ├── logging.js            # Request logging
│   ├── permission.js         # Permission checking
│   ├── cache_context.js      # Cache context management
│   ├── mobile_detect.js      # Mobile device detection
│   ├── get_ip.js             # Client IP extraction
│   └── [other middleware]
├── workers/                  # Background workers
│   ├── index.worker.js       # Main worker coordinator
│   ├── processor.worker.js   # Task processor
│   ├── queue.worker.js       # Queue management
│   ├── websocket.worker.js   # WebSocket handler
│   └── admin.worker.js       # Admin worker tasks
├── jobs/                     # Scheduled jobs (via node-schedule)
│   ├── cleanup.js            # Cleanup tasks (every 10 minutes)
│   ├── update_problems.js    # Problem updates (daily at midnight)
│   ├── crawl_judgement.js    # Judgement crawling (daily at noon)
│   └── warm_up.js            # Cache warming (every 5 minutes)
├── handlers/                 # Event handlers
│   ├── index.handler.js      # Main handler
│   ├── broadcast.benben.handler.js
│   ├── task_progress.benben.handler.js
│   └── [other handlers]
├── views/                    # Nunjucks templates
│   ├── layout.njk            # Main layout template

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laikit-dev/luogu-saver-legacy](https://github.com/laikit-dev/luogu-saver-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
