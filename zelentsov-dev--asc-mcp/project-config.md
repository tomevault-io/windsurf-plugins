---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Language
Response language: Russian
Comment language: English    

## Project Overview

MCP (Model Context Protocol) server for App Store Connect API integration, designed for Codex CLI. This server provides tools to manage iOS/macOS apps through App Store Connect.

## Build and Run Commands

```bash
# Build the project
swift build

# Run all unit tests
swift test

# Run the MCP server (requires environment variables or companies.json)
./.build/debug/asc-mcp

# Run with worker filtering (for clients with tool limits)
./.build/debug/asc-mcp --workers apps,builds,versions,reviews

# Run integration tests
./.build/debug/asc-mcp --test

# Clean build
swift package clean
```

## Environment Configuration

Three config methods (checked in priority order):
1. `--companies /path/to/companies.json` (CLI argument)
2. `ASC_MCP_COMPANIES` env var → path to JSON file
3. Default JSON: `~/.config/asc-mcp/companies.json`
4. `ASC_COMPANY_1_KEY_ID`, `ASC_COMPANY_2_KEY_ID`... (multi-company env vars)
5. `ASC_KEY_ID` + `ASC_ISSUER_ID` + `ASC_PRIVATE_KEY_PATH` (single company env vars)

Each company needs: `keyID`, `issuerID`, `privateKeyPath` (path to `.p8` file).

## Architecture

### Core Components

**WorkerManager** (`Workers/MainWorker/WorkerManager.swift`) — central registry, routes tool calls by prefix.

**Workers** (38 Swift worker classes; 34 `--workers` filter keys; 451 tools):

| Worker | Prefix | Tools | Domain |
|--------|--------|-------|--------|
| CompaniesWorker | `company_` | 3 | Multi-account management |
| AuthWorker | `auth_` | 4 | JWT tokens |
| AppsWorker | `apps_` | 9 | App listing, metadata, localizations |
| AccessibilityWorker | `accessibility_` | 6 | App Store accessibility declarations |
| WebhooksWorker | `webhooks_` | 11 | Webhook notifications, delivery diagnostics, receiver helpers |
| XcodeCloudWorker | `xcode_cloud_` | 30 | Xcode Cloud products, workflows, builds, artifacts, issues, test results, SCM |
| BuildsWorker | `builds_` | 4 | Build management |
| BuildBetaDetailsWorker | `builds_*_beta_` | 11 | TestFlight localizations, notifications, beta groups, individual testers |
| BuildProcessingWorker | `builds_*_processing_` | 4 | Build states, encryption |
| ExportComplianceWorker | `export_compliance_` | 11 | Encryption declarations, document delivery, build linkage, readiness |
| AppLifecycleWorker | `app_versions_` | 17 | Versions, age ratings, submit, release, phased rollout, delete |
| ReviewsWorker | `reviews_` | 8 | Customer reviews, responses, AI summarizations |
| BetaGroupsWorker | `beta_groups_` | 9 | TestFlight groups CRUD, testers, builds |
| BetaFeedbackWorker | `beta_feedback_` | 8 | TestFlight feedback screenshots, crash submissions, crash logs |
| InAppPurchasesWorker | `iap_` | 59 | IAP, versioned metadata, pricing, availability, offer codes, review assets |
| ProvisioningWorker | `provisioning_` | 17 | Bundle IDs, devices, certificates, profiles, capabilities |
| BetaTestersWorker | `beta_testers_` | 12 | Tester management, search, invite, relationships, invitations |
| AppInfoWorker | `app_info_` | 10 | App info, categories, localizations, EULA |
| PricingWorker | `pricing_` | 9 | Territories, availability, price points/schedule, app availabilities v2 |
| UsersWorker | `users_` | 10 | Team members, roles, invitations, visible apps |
| AppEventsWorker | `app_events_` | 9 | In-app events CRUD, localizations |
| AnalyticsWorker | `analytics_` | 11 | Sales/financial reports, app summary, analytics reports/instances/segments, snapshot status |
| SubscriptionsWorker | `subscriptions_` | 99 | Subscription and group versions, pricing, plan availability, offers, assets; includes offer-code, intro, promotional, and win-back sub-worker behavior |
| SandboxTestersWorker | `sandbox_` | 3 | Sandbox testers (list, update, clear purchase history) |
| BetaAppWorker | `beta_app_` | 10 | Beta app localizations, review submissions, review details |
| PreReleaseVersionsWorker | `pre_release_` | 3 | Pre-release versions (list, get, builds) |
| BetaLicenseAgreementsWorker | `beta_license_` | 3 | Beta license agreements (list, get, update) |
| ScreenshotsWorker | `screenshots_` | 16 | Screenshots, previews, sets, reorder, full upload, batch upload |
| CustomProductPagesWorker | `custom_pages_` | 10 | Custom product pages, versions, localizations |
| ProductPageOptimizationWorker | `ppo_` | 9 | A/B test experiments, treatments |
| PromotedPurchasesWorker | `promoted_` | 9 | Promoted in-app purchases; legacy image tools return migration guidance |
| ReviewAttachmentsWorker | `review_attachments_` | 4 | App Store review attachments (upload, get, delete, list) |
| ReviewSubmissionsWorker | `review_submissions_` | 9 | Generic App Store review submissions and submission items |
| MetricsWorker | `metrics_` | 4 | Performance/power metrics, diagnostics |

**Services**: HTTPClient (actor, GET/POST/PATCH/PUT/DELETE + retry with 429), JWTService (ES256), CompaniesManager

### Key Implementation Details

1. **Swift 6 Compliance**: All types `Sendable`, proper actor isolation
2. **JWT Auth**: CryptoKit ES256, tokens expire after 20 min

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zelentsov-dev/asc-mcp](https://github.com/zelentsov-dev/asc-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
