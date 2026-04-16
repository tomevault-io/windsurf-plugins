---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an IT infrastructure management system (DARI) that provides centralized user management, VPN access, Linux account provisioning, and LDAP-based authentication. The system manages Linux accounts, LDAP directory, VPN access with OTP, and user lifecycle management.

## Architecture

### Multi-Service Docker Architecture
- **Frontend**: SvelteKit application with SSR (port 3000 internally)
- **Backend**: Django + Django Ninja REST API (port 8080 internally)
- **Database**: PostgreSQL 13
- **LDAP**: OpenLDAP server for Linux authentication (port 636)
- **VPN**: OpenVPN server with OTP authentication (port 1194/udp)
- **Reverse Proxy**: Caddy server handling HTTPS and access control
- **Task Queue**: Celery with RabbitMQ for async tasks (user removal, deactivation)

### Key Integration Points
1. **LDAP Authentication**: All regular users authenticate via LDAP with passwords stored as SSHA hashes
2. **LDAP Sync**: Backend maintains LDAP directory synchronized with Django database for Linux PAM authentication
3. **VPN Auth**: OpenVPN uses openvpn-auth-ldap plugin to authenticate directly against LDAP server + PAM for OTP validation
4. **Session Management**: Frontend uses cookies for Django session authentication
5. **IP Whitelisting**: Caddy restricts most routes to configured networks (default: 10.125.0.0/16, 164.125.0.0/16)

### Data Flow
- **Registration**: User registers via `/api/register` → Backend atomically creates:
  - Django User + Profile + LinuxInfo
  - LDAP entry with SSHA-hashed password
  - Home directory with proper ownership
  - First user automatically becomes superuser
- **Login**: User logs in via frontend → Backend authenticates via LDAP (regular users) or Django auth (guest users) → Creates session
- **Password Management**: User changes password via `/api/password` → Backend updates LDAP password (SSHA hash)
- **VPN Setup**: User enables VPN → Backend generates Google Authenticator QR code → Stores in `/etc/qr/` → VPN server validates OTP + LDAP password

## Development Commands

### Environment Setup
```bash
# Copy and configure environment variables
cp .env.example .env
# Edit .env with required credentials (DB, LDAP, secrets)
```

### Development Mode (compose-dev.yml)
```bash
# Start all services with hot-reload and debug enabled
docker compose -f compose-dev.yml up --build

# Access frontend at http://localhost:8080
# Backend DEBUG=True, logs to /dev/null
# Volumes mount source code for hot-reload
```

### Production Mode (compose.yml)
```bash
# Start production services
docker compose up -d --build

# View logs
docker compose logs -f [service_name]

# Services: db, caddy, ldap, rabbitmq, celery_worker, celery_beat, vpn, backend, frontend
```

### Backend Development
```bash
# Enter backend container
docker compose exec backend bash

# Run Django migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Run Django shell
python manage.py shell

# Test Celery tasks
celery -A backend inspect active
```

### Frontend Development
```bash
# Enter frontend container (dev mode)
docker compose -f compose-dev.yml exec frontend sh

# Install dependencies
cd /app && pnpm install

# Build for production
pnpm run build

# Preview production build
pnpm run preview
```

### Database Operations
```bash
# Access PostgreSQL
docker compose exec db psql -U dari -d dari

# Backup database
docker compose exec db pg_dump -U dari dari > backup.sql

# Restore database
docker compose exec -T db psql -U dari dari < backup.sql
```

## Critical Backend Components

### [backend/backend/api.py](backend/backend/api.py)
Main API endpoints using Django Ninja. Key routes:
- `/api/register` - Register new user with Linux/LDAP account and password (creates everything atomically)
- `/api/login` - Authenticates via LDAP (regular users) or Django auth (guest users)
- `/api/password` - Change user's LDAP password (requires old password verification)
- `/api/qr` - Generates Google Authenticator for VPN OTP
- `/api/group` - Manages LDAP groups with member synchronization
- `/api/user` - Updates user attributes including password resets (admin only)
- `/api/guest` - Creates or updates guest user accounts (admin only)

### [backend/backend/utils.py](backend/backend/utils.py)
Core utilities:
- `LDAPOps` class - Manages LDAP operations (add/delete users/groups, password management)
  - `add_user()` - Creates LDAP entry with optional password (SSHA hash)
  - `set_password()` - Updates user's LDAP password
  - `authenticate_user()` - Validates username/password against LDAP
- `send_email()` - Bulk email to active users via SMTP/IMAP

### [backend/auth/models.py](backend/auth/models.py)
Django models:
- `Profile` - User profile with name, status (sta), expiration dates
- `LinuxInfo` - Linux account details (username, UID, GID, shell)
- `VPNInfo` - VPN/OTP enablement flag
- `GuestInfo` - Guest user metadata
- `LinuxGroup` - LDAP group with members
- `Server` - Managed servers with IP addresses

### [backend/auth/tasks.py](backend/auth/tasks.py)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pnucolab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
