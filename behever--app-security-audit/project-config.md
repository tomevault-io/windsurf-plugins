---
trigger: always_on
description: Comprehensive security audit for web apps, Supabase, databases, mobile apps, and APIs. Use when the user wants to audit security, check for vulnerabilities, review RLS policies, find exposed secrets, scan for XSS/CSRF/injection issues, review auth implementation, check API security, or harden their application. Covers: Supabase RLS and storage policies, web app XSS and CORS, mobile app secret storage, database access controls, API authentication and rate limiting, OWASP Top 10, dependency vulner
---


# App Security Audit

Comprehensive security audit workflow for web apps, Supabase projects, databases, mobile apps, and APIs.

## Quick Start

Run the automated scanner first, then deep-dive into areas with findings:

```bash
bash skills/app-security-audit/scripts/scan_project.sh /path/to/project
```

The scanner outputs a categorized report (CRITICAL / WARNING / INFO). Use it as a starting point — then follow the relevant sections below for manual review.

## Workflow Decision Tree

Determine what to audit based on the project type:

```
Is this a Supabase project?
├─ YES → Run steps 1, 2, 3, 5
└─ NO
   ├─ Web app? → Run steps 1, 3, 5
   ├─ Mobile app? → Run steps 1, 4, 5
   ├─ API/backend? → Run steps 1, 2, 5
   └─ Full stack? → Run ALL steps
```

## Step 1: Automated Scan

Run the scanner to catch common issues instantly:

```bash
bash skills/app-security-audit/scripts/scan_project.sh /path/to/project
```

**What it finds:**
- Hardcoded secrets (API keys, passwords, tokens)
- .env files in public directories
- XSS vectors (dangerouslySetInnerHTML, eval, innerHTML)
- Insecure token storage (localStorage)
- Service role key exposure
- Missing RLS in migrations
- Bare auth.uid() without subselect
- Dependency vulnerabilities (npm audit)
- CORS wildcard misuse
- Mobile-specific issues (AsyncStorage, ATS, cleartext)

Address all CRITICAL findings before proceeding.

## Step 2: Database & Supabase Deep Dive

> Reference: `references/supabase-security.md`, `references/database-security.md`

### RLS Audit

1. **List tables without RLS:**
   ```sql
   SELECT c.relname AS table_name, c.relrowsecurity AS rls_enabled
   FROM pg_class c
   JOIN pg_namespace n ON n.oid = c.relnamespace
   WHERE n.nspname = 'public' AND c.relkind = 'r'
   ORDER BY c.relname;
   ```

2. **Review all policies:**
   ```sql
   SELECT tablename, policyname, permissive, roles, cmd, qual, with_check
   FROM pg_policies WHERE schemaname = 'public'
   ORDER BY tablename, policyname;
   ```

3. **Check for performance issues:**
   - Bare `auth.uid()` → must be `(select auth.uid())`
   - Missing indexes on FK columns and RLS-referenced columns
   - JOINs in policy definitions (use subselects instead)

4. **Find missing FK indexes:**
   ```sql
   SELECT c.conrelid::regclass AS table_name, a.attname AS fk_column
   FROM pg_constraint c
   JOIN pg_attribute a ON a.attrelid = c.conrelid AND a.attnum = ANY(c.conkey)
   WHERE c.contype = 'f'
     AND NOT EXISTS (
       SELECT 1 FROM pg_index i
       WHERE i.indrelid = c.conrelid AND a.attnum = ANY(i.indkey)
     );
   ```

5. **Verify SECURITY DEFINER functions** have `SET search_path = public`

6. **Check anon role access** — what can unauthenticated users do?

7. **Storage bucket policies** — verify no public buckets allow unauth uploads

### Connection Security

- Connection strings only in env vars (never hardcoded)
- SSL/TLS enabled (`sslmode=require`)
- No `rejectUnauthorized: false` in production
- Database users follow least privilege

## Step 3: Web App Security Review

> Reference: `references/web-app-security.md`

### Priority Checks

1. **Auth token storage** — httpOnly cookies > localStorage
2. **XSS vectors** — dangerouslySetInnerHTML, innerHTML, eval, v-html
3. **API key exposure** — only anon/public keys in client bundles
4. **CORS** — no wildcard (*) with credentials
5. **CSP headers** — Content-Security-Policy configured
6. **Environment variables** — no secrets in NEXT_PUBLIC_/VITE_ prefixed vars

### Grep Patterns

```bash
# XSS
grep -rn 'dangerouslySetInnerHTML\|\.innerHTML\s*=\|\beval\s*(' --include='*.tsx' --include='*.ts' src/

# Auth storage
grep -rn 'localStorage.*token\|localStorage.*auth' --include='*.ts' --include='*.tsx' src/

# Exposed secrets
grep -rn 'NEXT_PUBLIC_.*SECRET\|NEXT_PUBLIC_.*PRIVATE' --include='*.ts' --include='*.env*' .

# CORS
grep -rn "Access-Control-Allow-Origin.*\*" --include='*.ts' --include='*.js' src/ server/
```

### Security Headers Checklist

Verify these headers are set (in middleware, next.config, or CDN):

| Header | Value |
|--------|-------|
| Content-Security-Policy | `default-src 'self'; script-src 'self'; frame-ancestors 'none'` |
| X-Frame-Options | `DENY` |
| X-Content-Type-Options | `nosniff` |
| Strict-Transport-Security | `max-age=31536000; includeSubDomains` |
| Referrer-Policy | `strict-origin-when-cross-origin` |
| Permissions-Policy | `camera=(), microphone=(), geolocation=()` |

## Step 4: Mobile App Security Review

> Reference: `references/mobile-app-security.md`

### Priority Checks

1. **Secure storage** — use expo-secure-store or react-native-keychain for secrets, NOT AsyncStorage
2. **Hardcoded secrets** — check app.json, app.config.js, and source for API keys
3. **Transport security** — iOS ATS enabled, Android cleartext disabled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [behever/app-security-audit](https://github.com/behever/app-security-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
