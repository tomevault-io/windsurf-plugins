---
trigger: always_on
description: This project creates a custom Model Context Protocol (MCP) server that provides Claude Code with up-to-date documentation for a specific Django + Vue.js technology stack. The server automatically fetches current versions and documentation for your libraries, including custom PyPI packages.
---

# Gojjo Django Vue MCP Documentation Server

## Overview

This project creates a custom Model Context Protocol (MCP) server that provides Claude Code with up-to-date documentation for a specific Django + Vue.js technology stack. The server automatically fetches current versions and documentation for your libraries, including custom PyPI packages.

## Problem Solved

**Claude Code Issues:**

- Suggests outdated library versions
- No knowledge of custom PyPI libraries (like `aida-permissions`)
- Generic context that includes unnecessary frameworks

**Our Solution:**

- Real-time documentation for your exact tech stack
- Current version information from PyPI/NPM
- Custom library documentation integration
- Automated updates via web scraping
- Shared team access via hosted server

## Technology Stack Covered

### Django Backend

- Django (latest stable)
- Django REST Framework (DRF)
- drf-spectacular
- django-cors-headers
- django-filter
- django-allauth
- djangorestframework-simplejwt
- stripe
- twilio
- django-storages[google]
- django-anymail[sendgrid]
- gunicorn
- psycopg[c]
- redis
- celery
- django-celery-beat
- flower
- pytest-django
- factory
- **aida-permissions** (custom library)
- pytest
- coverage
- factory-boy

### Vue.js Frontend

- Vue 3
- Vue Router
- Pinia (state management)
- Axios
- Tailwind CSS
- TipTap (rich text editor)
- Vite
- Jest
- Cypress
- vue-test-utils
- Vitest
- ESLint
- Playwright

## Architecture

```markdown
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Claude Code   │───▶│   MCP Server     │───▶│  Documentation  │
│                 │    │  (mcp.domain.com)│    │   Sources       │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        │
                       ┌──────────────────┐             │
                       │   Rate Limiting  │             │
                       │   & Caching      │             │
                       └──────────────────┘             │
                                                        ▼
                                              ┌─────────────────┐
                                              │ PyPI/NPM APIs   │
                                              │ GitHub Releases │
                                              │ Official Docs   │
                                              └─────────────────┘
```

## Implementation Plan

### Phase 1: Core MCP Server

1. **Basic MCP Server Structure**
   - Create `django_vue_mcp_server.py` with resource handlers
   - Implement PyPI/NPM API integration
   - Add caching layer for performance

2. **Documentation Fetching**
   - Real-time version checking
   - Custom library documentation (aida-permissions)
   - Integration examples and common patterns

3. **Local Testing**
   - Test MCP protocol communication
   - Verify Claude Code integration
   - Test all supported libraries

### Phase 2: Production Deployment

1. **Hetzner Server Setup**
   - Ubuntu server configuration
   - Docker containerization
   - Redis for caching

2. **Domain & SSL**
   - Configure `mcp.gojjoapps.com` subdomain
   - Let's Encrypt SSL certificate
   - Nginx reverse proxy

3. **HTTP API Wrapper**
   - REST endpoints for MCP resources
   - Health checks and monitoring
   - Webhook endpoints for auto-updates

### Phase 3: Security & Scaling

1. **Rate Limiting**
   - Anonymous: 100 requests/hour
   - API key: 1000 requests/hour
   - Open source projects: 5000 requests/hour

2. **Cost Protection**
   - Resource usage monitoring
   - Automatic scaling limits
   - Alert system for overages

3. **Open Source Preparation**
   - Documentation for self-hosting
   - API key system for public use
   - Community contribution guidelines

## File Structure

```markdown
django-vue-mcp-server/
├── CLAUDE.md                    # This file
├── README.md                    # Public documentation
├── requirements.txt             # Python dependencies
├── docker-compose.yml           # Production deployment
├── Dockerfile                   # Container configuration
├── nginx.conf                   # Reverse proxy config
├── systemd/                     # Service files
│   └── mcp-docs.service
├── src/
│   ├── django_vue_mcp_server.py    # Core MCP server
│   ├── production_mcp_server.py    # HTTP wrapper
│   ├── mcp_http_client.py          # Client for Claude Code
│   ├── auto_updater.py             # Automated doc updates
│   ├── rate_limiter.py             # Security layer
│   ├── auth.py                     # API key management
│   └── monitoring.py               # Usage tracking
├── docs/
│   ├── installation.md
│   ├── api.md
│   └── self-hosting.md
└── tests/
    ├── test_mcp_server.py
    ├── test_rate_limiting.py
    └── test_integrations.py
```

## Key Features

### 1. Real-Time Documentation

- Fetches current versions from PyPI/NPM APIs
- Scrapes release notes and changelog information
- Caches results for performance (6-hour refresh)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hmesfin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
