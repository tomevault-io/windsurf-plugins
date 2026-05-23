---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Architectural Constraints

- Always obey the decisions recorded in the Architecture Design Records directory (docs/adr). The user must approve any violations of these rules or decisions.
- This is a Google Apps Script project using the V8 runtime with clasp for local development
- All source code resides in the `src/` directory and uses `.gs` extensions for Google Apps Script files
- The system follows a pluggable agent architecture defined in ADR-004, with enhanced self-contained agent patterns (ADR-011)
- Generic service layer pattern (ADR-012) provides reusable functions for agent development

## Development Commands

### Account Setup and Management
```bash
# Initial Setup
npm run setup:account               # Interactive account configuration wizard
npm run switch:create-project-files  # Create project files for all accounts
npm run validate:accounts           # Validate account configuration
npm run auth:help                   # Show authentication guidance

# Status and Information
npm run switch:status               # Show all account statuses
```

### Deployment Operations (Streamlined)
```bash
# Core Deployment Commands (Push + Web App + Triggers)
npm run deploy:personal             # Complete deployment to personal account
npm run deploy:work                 # Complete deployment to work account
npm run deploy:all                  # Deploy to all configured accounts

# Monitoring and Logs
npm run logs:personal               # View execution logs from personal account
npm run logs:work                   # View execution logs from work account
npm run status:personal             # Check status of personal account
npm run status:work                 # Check status of work account

# Apps Script Editor Access
npm run open:personal               # Open Apps Script editor for personal account
npm run open:work                   # Open Apps Script editor for work account

# Web App URL Retrieval
npm run url:personal                # Get web app URL for personal account
npm run url:work                    # Get web app URL for work account
npm run url:all                     # Get web app URLs for all accounts
```

### Multi-Account Operations
```bash
# Batch Operations (All Accounts)
npm run status:all                  # Show status for all configured accounts
```

### Account Setup Process

#### First-Time Setup
1. **Account Configuration**: Run `npm run setup:account` to configure accounts
2. **Authentication**: Log into each account with `clasp --user [account] login`
3. **Project Files**: Run `npm run switch:create-project-files` to create clasp project files
4. **Validation**: Run `npm run validate:accounts` to verify setup
5. **Deploy**: Use `npm run deploy:[account]` for complete deployment

#### Account Configuration Format
The system uses `accounts.json` to manage multiple Google Apps Script deployments:
```json
{
  "defaultAccount": "work",
  "accounts": {
    "work": {
      "scriptId": "1Yyl2UjvQOBKxT1J6OXPzR0q5bpRBdLuE7MUgTuVdV7uUtdltIxQyQBK-",
      "description": "Red Hat",
      "webAppDeploymentId": "AKfycbx1L7phZrDzB699TRLDhSb5PCLbufYiGXcRU9ZPz2A"
    },
    "personal": {
      "scriptId": "1JvaGS8HDHIJoebhjY_2bQjUx0Tx2XVyHSJkaA5gV7_MEUWBixuRsHPno",
      "description": "Personal Gmail Account",
      "webAppDeploymentId": "AKfycbxH3nXXSOQ1teErs4nA8uojO2AI_qVLIeVY8HLHkkBv"
    }
  }
}
```

**Note**: The `webAppDeploymentId` field is automatically managed by the deployment script to maintain consistent web app URLs.

#### Authentication Requirements
Each account requires separate authentication:
```bash
clasp --user personal login    # Log into personal account
clasp --user work login        # Log into work account
```

#### Trigger Installation
**Important**: Automated trigger installation via `clasp run` is unreliable due to permission issues. Triggers must be installed manually:

1. Use `npm run open:personal` or `npm run open:work` to open Apps Script editor
2. Select trigger installation function from the dropdown:
   - `installTrigger` - Core email labeling trigger (hourly) - **Required**
   - `installSummarizerTrigger` - Email Summarizer trigger (daily) - Optional, only if using Email Summarizer
3. Click the Run button to install triggers
4. Grant necessary permissions when prompted

**Note**: The Reply Drafter no longer requires a separate trigger. It runs automatically as part of the hourly email processing via the dual-hook architecture (onLabel + postLabel).

The `deploy:[account]` commands attempt automated trigger installation but may fail on the `clasp run` portion.

#### Deployment Strategy Guide
- **Complete deployment**: Use `npm run deploy:[account]` for full system deployment (code + web app + triggers)
- **All accounts**: Use `npm run deploy:all` for batch deployment to all configured accounts
- **The deployment script uses --force flag** to prevent "skipping push" issues
- **Smart web app deployment**: Maintains consistent URLs by redeploying to existing deployments rather than creating new ones

## Architecture Overview

### Core Components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghelleks/email-agent](https://github.com/ghelleks/email-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
