---
trigger: always_on
description: A parent activity app for connecting children and organizing activities. Built with React frontend and Node.js/PostgreSQL backend.
---

# Cluubz Parent Activity App - Claude Context

## Project Overview
A parent activity app for connecting children and organizing activities. Built with React frontend and Node.js/PostgreSQL backend.

## ✅ CORRECT PROJECT STRUCTURE:
- **Working Directory**: `/home/jonathan/Claud-Clubs2/cluubz-web/` (ONLY THIS ONE)
- **Dead Projects**: `parent-activity-web` (DO NOT USE - being decommissioned)

## Key URLs
- **Staging Frontend**: https://cluubz-staging.web.app
- **Staging Backend**: https://cluubz-backend-staging-294249cc77d1.herokuapp.com
- **Production Frontend**: https://cluubz-production.web.app
- **Production Backend**: https://cluubz-backend-production-ddf3bc671afb.herokuapp.com

## Test Users
- **new700@example.com** - Current test user
- **testxy@example.com** - Used for skeleton connection testing

## 🔍 BROWSER TESTING & QA

### Playwright MCP for Automated Testing

Claude Code has access to the **Playwright Model Context Protocol (MCP)** for automated browser testing and quality assurance.

**Available Application Environments:**
- **Staging App**: https://cluubz-staging.web.app (primary testing environment)
- **Production App**: https://cluubz-production.web.app (use with caution)

**Browser Testing Capabilities:**
- ✅ **Navigate**: Open Chrome and navigate to the staging/production app
- ✅ **Interact**: Click links, buttons, input text into forms
- ✅ **Validate**: Take screenshots and verify UI rendering
- ✅ **Debug**: Check JavaScript console for runtime errors
- ✅ **Login**: Test authentication flows with test users
- ✅ **E2E Testing**: Perform end-to-end user workflows

**Example Testing Workflows:**
```
1. Login Test:
   - Navigate to https://cluubz-staging.web.app
   - Enter test user credentials (new700@example.com)
   - Verify successful login and dashboard display

2. Feature Validation:
   - Test specific features after deployment
   - Take screenshots to verify UI changes
   - Check console for JavaScript errors

3. Regression Testing:
   - Verify existing functionality still works
   - Test common user workflows
   - Validate critical paths (login, create activity, connections)
```

**When to Use Browser Testing:**
- ✅ After deploying changes to staging
- ✅ Before deploying to production
- ✅ When implementing new features
- ✅ When fixing UI bugs
- ✅ When user reports issues

**Test User Credentials for Login:**
- Email: **new700@example.com**
- Password: (user will provide if needed for testing)

## 🚨 CRITICAL DATABASE SAFETY RULES

### ⚠️ MANDATORY ENVIRONMENT PROTECTION FOR CLAUDE:

**BEFORE ANY DATABASE OPERATION, CLAUDE MUST:**
1. ✅ **VERIFY DATABASE URL**: Always confirm which database URL is being used
2. ✅ **MATCH REQUEST TO ENVIRONMENT**: Ensure the operation targets the correct environment
3. ✅ **VALIDATE ENVIRONMENT VARIABLES**: Never assume DATABASE_URL points to the right environment
4. ✅ **EXPLICIT CONFIRMATION**: When user says "production", use ONLY production URLs
5. ✅ **NO ASSUMPTIONS**: Never run scripts without explicitly setting the correct DATABASE_URL

### 🔒 ENVIRONMENT DATABASE URLS - CLAUDE MANDATORY REFERENCE:

```bash
# STAGING ONLY - cluubz-backend-staging
STAGING_DATABASE_URL="postgres://uf589tiejgjbfq:p56aef075a3fc09891e7550eaac1c8afaf4604985023985df9e0dd004340d5d8a@c7itisjfjj8ril.cluster-czrs8kj4isg7.us-east-1.rds.amazonaws.com:5432/daha3hutf78prh"

# PRODUCTION ONLY - cluubz-backend-production
PRODUCTION_DATABASE_URL="postgres://uagekj91u4hrmc:p904b26b59665bc319897558fbcceda73908268f3ffa579f13acb6ea29ab2b74a@c7itisjfjj8ril.cluster-czrs8kj4isg7.us-east-1.rds.amazonaws.com:5432/denpo4iq2ebe9i"
```

### 🚨 CLAUDE DATABASE OPERATION PROTOCOL:

**NEVER RUN DATABASE SCRIPTS WITHOUT:**
1. Setting explicit DATABASE_URL for the target environment
2. Double-checking which environment the user requested
3. Confirming the database URL matches the request (staging vs production)

**CORRECT EXAMPLES:**
```bash
# When user says "clean production clubs":
DATABASE_URL="postgres://uagekj91u4hrmc:..." node clean-production-clubs.js

# When user says "clean staging clubs":
DATABASE_URL="postgres://uf589tiejgjbfq:..." node clean-staging-clubs.js
```

**FORBIDDEN - NEVER DO THIS:**
```bash
# WRONG - relies on environment variable that could be anything
node clean-production-clubs.js
```

## 🚀 DEPLOYMENT PIPELINE

### 🚨 CRITICAL DEPLOYMENT SAFETY FOR CLAUDE:

**BEFORE ANY DEPLOYMENT, CLAUDE MUST:**
1. ✅ **VERIFY TARGET ENVIRONMENT**: Confirm staging vs production request
2. ✅ **VERIFY DEPLOYMENT COMMANDS**: Use correct Heroku app and Firebase project
3. ✅ **VERIFY BRANCH**: Ensure pushing to correct git remote
4. ✅ **DOUBLE-CHECK URLS**: Confirm deployment targets match user request

### 🔒 DEPLOYMENT COMMAND MAPPING - CLAUDE MANDATORY REFERENCE:

**⚠️ CRITICAL: Always deploy from the ROOT directory `/home/jonathan/Claud-Clubs2/`**

#### 🚨 MANDATORY: ONLY Use Deployment Scripts - NO MANUAL COMMANDS ALLOWED

**CLAUDE PRE-DEPLOYMENT CHECKLIST (CHECK EVERY TIME):**
- [ ] Am I about to run `npm run build` directly? **STOP - USE SCRIPT INSTEAD**
- [ ] Am I about to run `firebase deploy` directly? **STOP - USE SCRIPT INSTEAD**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonvic1891) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
