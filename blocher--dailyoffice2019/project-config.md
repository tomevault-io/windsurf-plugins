---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected
---

# Daily Office 2019 - Developer Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected
information that does not match the info here.

## Project Overview

Daily Office 2019 is a Django + Vue.js web application for Christian daily prayer services.

**Architecture:**

- **Backend**: Django 5.2+ in `/site/` directory with PostgreSQL database
- **Frontend**: Vue 3 + Vite in `/app/` directory with TypeScript support
- **Mobile**: Capacitor for iOS/Android apps
- **Deployment**: Deploys to Cloudflare for static front end. Uses git deploy hooks to deploy API; no longer uses django-distill

## Critical Setup Requirements

### Prerequisites - Install These First

- Python 3.13
- Node.js 24.4+ (tested with Node 20)
- PostgreSQL 17.5+
- Memcached 1.6+

**On Ubuntu/Debian:**

```bash
sudo apt-get update
sudo apt-get install -y postgresql memcached python3-venv python3-pip
```

## Working Effectively

### Bootstrap Environment - ALWAYS Do This First

```bash
# 1. Set up environment files
cp app/.env.development app/.env.local
cp site/website/.env.example site/website/.env

# 2. Edit site/website/.env - Set ALL required environment variables
# Add development values for all missing variables:
DEBUG=True
SECRET_KEY=development-secret-key-not-for-production
GOOGLE_API_KEY=development-api-key
GOOGLE_CUSTOM_SEARCH_ENGINE_KEY=development-search-key
OPENAI_API_KEY=development-openai-key
# ... (see Complete Environment Setup section below)

# 3. Start required services
sudo service postgresql start
sudo service memcached start
```

### Database Setup - Takes ~4 seconds

```bash
# Create database and user
sudo -u postgres psql -c "CREATE DATABASE dailyoffice;"
sudo -u postgres psql -c "CREATE USER dailyoffice WITH PASSWORD 'password';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE dailyoffice TO dailyoffice;"

# Import database dump - Takes 3-4 seconds
unzip -p site/dailyoffice_2024_01_30.sql.zip dailyoffice_2024_01_30.sql | sudo -u postgres psql dailyoffice
```

### Python Environment Setup - NETWORK ISSUES EXPECTED

```bash
cd site
python3 -m venv env
source env/bin/activate

# CRITICAL: Network timeouts are common with PyPI
# NEVER CANCEL: Takes 5-45 minutes when working, use maximum timeouts
pip install --timeout 1200 --retries 10 -r requirements.txt
```

**EXPECTED ISSUE**: `pip install` frequently fails with `ReadTimeoutError` due to network issues. **NEVER CANCEL** -
retry with longer timeouts.

**Workaround if pip install fails completely:**

```bash
# Install core dependencies individually with retries
pip install --timeout 600 --retries 5 Django==5.2 psycopg-binary beautifulsoup4 requests arrow django-environ
```

### Backend Node.js Setup - Takes ~2 minutes

```bash
cd site
npm install
# Ignore security warnings - Takes 1.5-2 minutes, NEVER CANCEL
```

### Frontend Setup - FONTAWESOME PRO AUTHENTICATION REQUIRED

```bash
cd app
npm install
```

**EXPECTED ISSUE**: Fails with `ENOTFOUND npm.fontawesome.com` due to FontAwesome Pro dependencies requiring
authentication.

**SOLUTION**: Configure FontAwesome Pro credentials using GitHub Personal Access Token:

1. **Create GitHub Personal Access Token**:
   - Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Generate new token with `read:packages` scope
   - Copy the token

2. **Configure npm for FontAwesome Pro**:
   ```bash
   # Set FontAwesome Pro registry
   npm config set "@fortawesome:registry" https://npm.fontawesome.com/
   
   # Configure authentication using GitHub token
   npm config set "//npm.fontawesome.com/:_authToken" YOUR_GITHUB_TOKEN
   
   # Or use FontAwesome Pro token directly (if you have one)
   npm config set "//npm.fontawesome.com/:_authToken" YOUR_FONTAWESOME_PRO_TOKEN
   ```

3. **Alternative: Use .npmrc file**:
   Create `app/.npmrc` with:
   ```
   @fortawesome:registry=https://npm.fontawesome.com/
   //npm.fontawesome.com/:_authToken=${FONTAWESOME_TOKEN}
   ```

4. **Environment variable approach**:
   ```bash
   export FONTAWESOME_TOKEN=your_github_token_or_fontawesome_pro_token
   cd app && npm install
   ```

**Note**: FontAwesome Pro requires either a GitHub Personal Access Token with `read:packages` scope or a direct FontAwesome Pro token. Contact project maintainers if you don't have access to either.

## Build Process - NEVER CANCEL BUILDS

### Backend Django Build

```bash
cd site
source env/bin/activate

# Collect static assets - Takes 30-60 seconds
python manage.py collectstatic --noinput

# DO NOT Generate static site 

```

```bash
# Clean and build everything - NEVER CANCEL, use long timeouts
make clean build
```

## Development Servers

### Django API Server

```bash
cd site
source env/bin/activate
python manage.py runsslserver
# Accessible at https://127.0.0.1:8000/
# API docs at https://127.0.0.1:8000/api/
```

**Note**: Requires `django-sslserver` package (part of requirements.txt)

### Frontend Development (When FontAwesome Issues Resolved)

```bash
cd app
npm run dev
# Accessible at http://127.0.0.1:8080
```

## Complete Environment Setup

**CRITICAL**: The .env file requires ALL these variables for Django to start:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blocher/dailyoffice2019](https://github.com/blocher/dailyoffice2019) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
