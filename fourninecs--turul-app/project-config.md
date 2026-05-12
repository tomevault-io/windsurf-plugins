---
trigger: always_on
description: **Recommended approach** (most reliable):
---

# Turul - Development Notes

## Running the App

### Electron (Desktop) Mode
**Recommended approach** (most reliable):
```bash
npm run dev:simple
```

Or manually:
```bash
npm run dev:vite &
npx electron .
```

**Why `npm run dev` may not work:**
- Uses `wait-on` which can get stuck; `dev:simple` uses a sleep delay instead

## Build Commands

```bash
npm run build              # Compile Vite + Electron (runs build:vite then build:electron)
npm run package            # Current platform
npm run package:mac        # macOS
npm run package:win        # Windows
npm run package:linux      # Linux
```

## Project Structure

```
src/
├── main/                          # Electron main process (Node.js)
│   ├── index.ts                   # Electron entry point
│   ├── preload.ts                 # IPC bridge (contextIsolation)
│   ├── auth/                      # Auth service
│   │   ├── auth-service.ts        # Password-based auth, session management
│   │   ├── biometric-service.ts   # Touch ID (macOS)
│   │   ├── crypto-service.ts      # AES-256-GCM encryption/decryption
│   │   └── app-profile-manager.ts # App-level profile persistence
│   ├── ai/                        # AI Chat backend
│   │   ├── ai-service.ts          # Tool loop, streaming via AsyncGenerator
│   │   ├── ai-provider.ts         # Provider interface
│   │   ├── system-prompt.ts       # System prompt
│   │   ├── providers/             # bedrock-provider.ts (primary)
│   │   └── tools/                 # aws-tools.ts, db-tools.ts, gcp-tools.ts, tool-registry.ts
│   ├── aws/
│   │   ├── client-factory.ts      # AWS SDK v3 client management (121 clients)
│   │   ├── profile-manager.ts     # AWS profile/credential management
│   │   ├── rate-limiter.ts        # Per-service API rate limiting
│   │   ├── scanners/              # 117 service scanner files
│   │   ├── iam-analysis/          # IAM security analysis (6 files: unused-roles, overly-permissive, cross-account-trust, password-policy, types, index)
│   │   ├── network-analysis/      # EC2/RDS reachability (reachability.ts, types.ts, index.ts)
│   │   ├── discovery/             # cost-explorer.ts, cost-resource-checks.ts, discovery-engine.ts, tagging-api.ts
│   │   ├── security/
│   │   │   ├── security-hub.ts    # Security Hub integration
│   │   │   ├── best-practices/    # EC2, S3, IAM, RDS, CloudTrail, KMS, VPC checks + index + types
│   │   │   └── compliance/        # CIS AWS v3 framework (cis-controls.ts, index.ts, types.ts)
│   │   └── well-architected/      # Workloads, lens reviews, improvements, best-practices/, types, index
│   ├── gcp/
│   │   ├── client-factory.ts      # GCP SDK client management (36 imports, 59 getters)
│   │   ├── auth-manager.ts        # gcloud ADC login/revoke via CLI subprocess
│   │   ├── gcloud-resolver.ts     # Cross-platform gcloud binary path resolver (cached, DB-backed)
│   │   ├── project-manager.ts     # Multi-project support
│   │   ├── scanners/              # 85 GCP service scanners
│   │   ├── assessment/            # Multi-dimensional scoring (orchestrator.ts, scoring.ts, types.ts, index.ts)
│   │   ├── iam-analysis/          # Service account analysis (unused-service-accounts.ts, overly-permissive.ts, service-account-keys.ts, cross-project-bindings.ts, types.ts, index.ts)
│   │   ├── label-governance/      # Label compliance pipeline (index.ts, types.ts)
│   │   ├── network-analysis/      # VPC reachability (reachability.ts, vpc-analysis.ts, types.ts, index.ts)
│   │   ├── cost/                  # Billing, CUD, recommendations, GKE costs, idle resources, stopped VMs (12 files)
│   │   ├── security/
│   │   │   ├── scc-integration.ts # Security Command Center
│   │   │   ├── best-practices.ts  # GCP best practices checks
│   │   │   └── compliance/        # CIS GCP framework (cis-gcp-controls.ts, index.ts)
│   │   └── well-architected/      # 5 pillar check files + index + types (GCP-native, no API dependency)
│   ├── database/
│   │   └── db-manager.ts          # SQLite (better-sqlite3, WAL mode, 26 migrations)
│   ├── health/
│   │   └── environment-checker.ts # Validates gcloud, AWS CLI, credentials at startup
│   ├── scanning/                  # scan-orchestrator.ts, gcp-scan-orchestrator.ts, multi-account-orchestrator.ts, relationship-builder.ts, gcp-relationship-builder.ts, scan-scheduler.ts, scan-diff.ts
│   ├── assessment/                # AWS assessment (orchestrator.ts, scoring.ts, recommendations.ts, index.ts)
│   ├── reports/                   # assessment-pdf-generator.ts, gcp-assessment-pdf-generator.ts, cost-export.ts, cost-pdf-generator.ts, csv-generator.ts, excel-generator.ts, gke-cost-export.ts, gke-cost-pdf-generator.ts, optimization-export.ts, optimization-pdf-generator.ts, pdf-chart-helpers.ts, pdf-generator.ts
│   └── ipc/                       # 10 Electron IPC handler files: app-handlers.ts, auth-handlers.ts, aws-handlers.ts, chat-handlers.ts, gcp-handlers.ts, profile-handlers.ts, resource-handlers.ts, ipc-utils.ts, validation.ts, index.ts
├── renderer/                      # React frontend (Vite)
│   ├── App.tsx                    # Router + sidebar (22 nav items filtered by provider; Settings/Profiles/Lock in sidebar footer)
│   ├── main.tsx                   # React entry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FournineCS/turul-app](https://github.com/FournineCS/turul-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
