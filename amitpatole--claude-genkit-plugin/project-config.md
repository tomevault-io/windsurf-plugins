---
trigger: always_on
description: This repository uses **autonomous agents** that run 24/7 to manage deployments, monitor health, and maintain the codebase.
---

# Autonomous Agents Documentation

This repository uses **autonomous agents** that run 24/7 to manage deployments, monitor health, and maintain the codebase.

## Overview

Two primary agents operate continuously:

1. **Schedule Enforcement Agent** - Ensures all deployments occur during the approved time window
2. **Monitoring & Maintenance Agent** - Monitors health, fixes issues, and manages updates

## Agent 1: Schedule Enforcement Agent

### Purpose
Enforces strict deployment schedules to ensure all pushes, releases, and deployments occur only during the approved time window.

### Schedule
- **Runs:** Every 5 minutes (24/7)
- **Workflow:** `.github/workflows/schedule-enforcement-agent.yml`

### Deployment Window
**ALLOWED:** 10:00 PM - 8:00 AM EST
**BLOCKED:** 8:00 AM - 10:00 PM EST

### Features
- ✅ Checks all deployment workflows for schedule compliance
- ✅ Blocks deployments outside the allowed window
- ✅ Monitors recent workflow runs for violations
- ✅ Generates violation reports
- ✅ Provides real-time status updates
- ✅ Heartbeat monitoring to confirm agent is running

### Actions Taken
1. **Pre-deployment Check:** Every deployment workflow first checks if current time is within the window
2. **Block on Violation:** Exits with error code 1 if deployment attempted outside window
3. **Generate Reports:** Creates violation reports in `agent-reports/` directory
4. **Status Updates:** Updates `.github/agent-status/schedule-enforcement.txt` every 5 minutes

### How It Works
```bash
# Check if within deployment window (10 PM - 8 AM EST)
CURRENT_HOUR=$(TZ='America/New_York' date +%H)

# Hours 22-23 (10 PM - 11:59 PM) OR 0-7 (12 AM - 7:59 AM) = ALLOWED
# Hours 8-21 (8 AM - 9:59 PM) = BLOCKED

if [ "$CURRENT_HOUR" -ge 22 ] || [ "$CURRENT_HOUR" -lt 8 ]; then
  echo "✅ Within deployment window - Proceeding"
else
  echo "🚫 BLOCKED: Outside deployment window"
  exit 1
fi
```

### Modified Workflows
All deployment workflows now include schedule enforcement:
- `publish-vscode-extension.yml`
- `vscode-extension-ci.yml` (publish job)
- `vscode-extension-release.yml`

### Monitoring
Check agent status at any time:
- **Status File:** `.github/agent-status/schedule-enforcement.txt`
- **Violation Reports:** `agent-reports/violation-*.md`
- **GitHub Actions:** View runs of "Schedule Enforcement Agent (24x7)"

---

## Agent 2: Monitoring & Maintenance Agent

### Purpose
Continuously monitors the extension and plugin, automatically fixes issues, manages updates, and responds to user reports.

### Schedule
- **Runs:** Every 10 minutes (24/7)
- **Workflow:** `.github/workflows/monitoring-maintenance-agent.yml`
- **Also triggers on:** New issues, pull requests

### Features

#### 1. Health Monitoring
- ✅ Compilation checks (TypeScript → JavaScript)
- ✅ package.json validation
- ✅ Security vulnerability scanning
- ✅ Marketplace metrics tracking
- ✅ Issue/PR monitoring

#### 2. Automatic Fixes
- 🔧 **Compilation Failures:** Reinstalls dependencies, cleans build artifacts
- 🔧 **Security Vulnerabilities:** Runs `npm audit fix` automatically
- 🔧 **Outdated Dependencies:** Updates patch versions safely
- 🔧 **Known Issues:** Applies automatic fixes when patterns are detected

#### 3. Issue Management
- 💬 **Auto-responds** to bug reports with triage information
- 💬 **Auto-labels** issues (bug, enhancement, needs-triage)
- 💬 **Tracks** critical issues and alerts maintainers
- 💬 **Provides** troubleshooting guidance to users

#### 4. Feature Research
- 🔍 Tracks latest Firebase Genkit releases
- 🔍 Monitors feature requests
- 🔍 Analyzes marketplace trends
- 🔍 Identifies popular user requests

#### 5. Dependency Management
- 📦 Checks for outdated dependencies
- 📦 Auto-updates patch versions (safe updates)
- 📦 Stages updates for deployment window
- 📦 Generates update reports

### Actions Taken

#### Automatic Fixes (Staged for Deployment Window)
All automated fixes respect the deployment schedule:
```bash
CURRENT_HOUR=$(TZ='America/New_York' date +%H)
if [ "$CURRENT_HOUR" -ge 22 ] || [ "$CURRENT_HOUR" -lt 8 ]; then
  git push  # Push immediately during window
else
  echo "⏰ Staged for deployment window (10 PM - 8 AM EST)"
fi
```

#### Compilation Failure Response
```bash
1. Detects compilation failure
2. Removes node_modules and out directories
3. Reinstalls dependencies (npm install)
4. Recompiles (npm run compile)
5. If successful: Commits and pushes fix
6. If failed: Creates issue for manual review
```

#### Security Vulnerability Response
```bash
1. Detects vulnerabilities via npm audit
2. Runs npm audit fix
3. Tests compilation after fix
4. If successful: Commits and stages for deployment window
5. If failed: Creates high-priority issue
```

#### Issue Auto-Response
When a new issue is opened:
- **Bug reports:** Auto-labels, provides troubleshooting steps, triages
- **Feature requests:** Auto-labels, explains review process, gathers feedback
- **Questions:** Provides links to documentation

### Monitoring
- **Status File:** `.github/agent-status/maintenance-agent.txt`
- **Health Reports:** `agent-reports/maintenance-*.md`
- **GitHub Actions:** View runs of "Monitoring & Maintenance Agent (24x7)"

---

## Agent Status Monitoring

### Real-Time Status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amitpatole/claude-genkit-plugin](https://github.com/amitpatole/claude-genkit-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
