---
trigger: always_on
description: > **Note for AI Assistants**: This is a frontend-specific context file for the LiteMaaS React application. For project overview, see root CLAUDE.md. For backend context, see backend/CLAUDE.md.
---

# CLAUDE.md - LiteMaaS Frontend Context

> **Note for AI Assistants**: This is a frontend-specific context file for the LiteMaaS React application. For project overview, see root CLAUDE.md. For backend context, see backend/CLAUDE.md.

## 🎯 Frontend Overview

**@litemaas/frontend** - React 18 application with TypeScript, Vite, and PatternFly 6 component library.

**Development Server**: Running on port 3000 with Vite HMR (Hot Module Replacement) and auto-refresh

## 🚨 CRITICAL FOR AI ASSISTANTS - Server and Logging

**⚠️ The frontend dev server is already running!** Do not start new processes.

### Checking Frontend Status and Logs

```bash
# DO NOT run npm run dev - server is already running!

# Check recent frontend logs (last 100 lines):
tail -n 100 ../logs/frontend.log

# Watch frontend logs in real-time:
tail -f ../logs/frontend.log

# Check for compilation errors:
grep -i "error\|failed" ../logs/frontend.log | tail -n 20

# Check for warnings (React, deprecations):
grep -i "warning" ../logs/frontend.log | tail -n 20

# Check Vite HMR updates:
grep "hmr" ../logs/frontend.log | tail -n 20

# Verify server is responding:
curl http://localhost:3000
```

### Server Information

- **Dev Server URL**: `http://localhost:3000`
- **HMR**: Enabled - changes to components instantly reflect in browser
- **Auto-refresh**: Browser automatically updates on file save
- **Log Location**: `../logs/frontend.log` (relative to frontend directory)
- **Build Output**: Check logs for TypeScript/ESLint errors

### Debugging Workflow

1. **Make component changes** - Save the file
2. **Check logs for compilation** - `tail -n 50 ../logs/frontend.log`
3. **If TypeScript errors** - Fix types and save, Vite will recompile
4. **If ESLint warnings** - Fix or add disable comment if intentional
5. **Check browser** - HMR should auto-update, check browser console for runtime errors
6. **If HMR fails** - Browser will show error overlay with details

### Common Frontend Log Patterns

```bash
# Check for failed API calls:
grep -i "axios\|fetch\|401\|403\|404\|500" ../logs/frontend.log | tail -n 20

# Check for React errors:
grep -i "react\|hook\|render\|component" ../logs/frontend.log | tail -n 20

# Check for PatternFly issues:
grep -i "patternfly\|pf-v6" ../logs/frontend.log | tail -n 20

# Check for build/bundle issues:
grep -i "vite\|rollup\|bundle\|chunk" ../logs/frontend.log | tail -n 20
```

## 📁 Frontend Structure

See [`docs/architecture/project-structure.md`](../docs/architecture/project-structure.md) for complete frontend directory structure.

## 🎨 PatternFly 6 Critical Requirements

⚠️ **MANDATORY**: Follow the [PatternFly 6 Development Guide](../docs/development/pf6-guide/README.md) as the **AUTHORITATIVE SOURCE** for all UI development.

### Essential Rules

1. **Class Prefix**: ALL PatternFly classes MUST use `pf-v6-` prefix
2. **Design Tokens**: Use semantic tokens only, never hardcode colors
3. **Component Import**: Import from `@patternfly/react-core` v6 and other @patternfly libraries
4. **Theme Testing**: Test in both light and dark themes
5. **Table Patterns**: Follow guide's table implementation (current code may be outdated)

### Common Mistakes and Token Usage

**Critical rules** - See [`docs/development/pf6-guide/guidelines/styling-standards.md`](../docs/development/pf6-guide/guidelines/styling-standards.md) for complete guide:

- ✅ ALWAYS use `pf-v6-` prefix for component classes
- ✅ ALWAYS use `--pf-t--` prefix for design tokens (semantic tokens with `-t-`)
- ✅ Choose tokens by meaning (e.g., `--pf-t--global--color--brand--default`), not appearance
- ❌ NEVER hardcode colors or measurements
- ❌ NEVER use legacy `--pf-v6-global--` tokens or numbered base tokens

## 🗃️ State Management

**React Context**:

- **AuthContext** - Authentication state (user, roles, isAuthenticated)
- **NotificationContext** - App-wide notification system
- **BrandingContext** - Branding settings from `/api/v1/branding` endpoint (5-min stale time, fallback to defaults)
- **ConfigContext** - Application configuration from `/api/v1/config` endpoint
  - **Base Config**: `usageCacheTtlMinutes`, `version`, `environment`
  - **Admin Analytics Config**: All UI-relevant admin analytics settings (pagination, limits, thresholds)
  - Integrates with React Query `staleTime`: `config.usageCacheTtlMinutes * 60 * 1000`
  - Pattern: Dynamic cache TTL eliminates hardcoded values in query hooks

**React Query**: Server state management with dynamic stale time from ConfigContext, 10min cache time, 3 retries

### Admin Analytics Configuration

**Hook**: `useAdminAnalyticsConfig()` provides pagination limits, date range limits, trend thresholds, and export limits from backend.

**Integration**: Dynamic configuration eliminates hardcoded values, integrates with React Query `staleTime` via ConfigContext.

**Admin Component Structure**:

- `components/admin/` - Admin-specific UI components
  - `MetricsOverview.tsx` - Shared usage analytics dashboard with trend indicators and auth-aware admin sections
  - `TopUsersTable.tsx` - Admin-only user usage breakdown table rendered by `MetricsOverview`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rh-aiservices-bu/litemaas](https://github.com/rh-aiservices-bu/litemaas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
