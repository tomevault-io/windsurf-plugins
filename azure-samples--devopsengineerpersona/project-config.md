---
trigger: always_on
description: Multilingual gift exchange app built with **React 19 + Vite 8** frontend and **Azure Functions v4** API, deployed on **Azure Static Web Apps** with **Cosmos DB**. Supports 9 languages, PWA, dark mode, QR code sharing, calendar integration, and hardened security.
---

# Copilot Instructions for Zava Gift Exchange

## Project Overview
Multilingual gift exchange app built with **React 19 + Vite 8** frontend and **Azure Functions v4** API, deployed on **Azure Static Web Apps** with **Cosmos DB**. Supports 9 languages, PWA, dark mode, QR code sharing, calendar integration, and hardened security.

## Tech Stack
| Layer | Technology |
|-------|-----------|
| Frontend | React 19, Vite 8, Tailwind CSS 4, shadcn/ui, Framer Motion |
| API | Azure Functions v4, TypeScript 5.9, Node.js 20 |
| Database | Azure Cosmos DB (Serverless / Free Tier) |
| Email | Azure Communication Services (optional) |
| Monitoring | Azure Application Insights (frontend + backend) |
| Infrastructure | Bicep (IaC), Azure Static Web Apps |
| CI/CD | GitHub Actions (5 workflows) |
| Testing | Jest (332 unit tests), Playwright (E2E, 3 browsers) |

## Environment Strategy
| Environment | Resource Group | SWA | Cosmos DB | Email | Lifecycle |
|-------------|---------------|-----|-----------|-------|-----------|
| PR | `ZavaGiftExchange-pr-{n}` | Standard | Serverless | ❌ | Ephemeral (auto-deleted on PR close) |
| QA | `ZavaGiftExchange-qa` | Standard | Serverless | ✅ | Persistent |
| Production | `ZavaGiftExchange` | Standard | Serverless | ✅ | Persistent |

## Project Structure
```
├── src/                        # React frontend
│   ├── components/             # 18 view/UI components
│   │   └── ui/                 # shadcn/ui primitives
│   ├── lib/                    # Types, API client, utils, calendar, currency
│   │   └── translations/       # 9 language files (en, es, pt, fr, it, ja, zh, de, nl)
│   ├── hooks/                  # useDarkMode, useLocalStorage, useMobile
│   └── styles/                 # CSS theme (light/dark)
├── api/                        # Azure Functions backend
│   └── src/
│       ├── functions/          # 8 HTTP endpoints
│       ├── shared/             # Cosmos DB, email, telemetry, rate limiter, types
│       └── __tests__/          # Jest tests (332 tests, 15 suites)
├── e2e/                        # Playwright E2E tests
├── tests/load/                 # Azure Load Testing (JMeter + config)
├── infra/                      # Bicep templates + parameter files
│   └── modules/                # Monitoring alerts (Bicep module)
├── scripts/                    # deploy.sh, deploy.ps1, e2e-summary.sh, setup, type validation
├── .devcontainer/              # Dev container / Codespaces config
├── .github/
│   ├── copilot-instructions.md # Global project context (this file)
│   ├── copilot-constitution.md # Non-negotiable rules for all agents
│   ├── copilot-memory.md       # Architecture decisions & known gotchas
│   ├── agents/                 # 6 specialized Copilot agents
│   ├── specs/                  # Specification templates & examples
│   ├── prompts/                # Reusable prompt library
│   ├── CODEOWNERS              # File ownership mapping
│   └── workflows/              # CI/CD, cleanup, CodeQL, dependency review, load testing
├── docs/
│   ├── runbooks/               # SRE runbooks (error rate, DB outage, rollback)
│   └── *.md                    # Developer guides
├── docker-compose.yml          # Cosmos DB + Azurite emulators
└── staticwebapp.config.json    # SWA routing, CSP headers, security
```

## API Routes
| Method | Route | Purpose |
|--------|-------|---------|
| `POST` | `/api/games` | Create game (validates date, 3+ participants) |
| `GET` | `/api/games/{code}` | Get game (access-controlled by token) |
| `PATCH` | `/api/games/{code}` | Update game (organizer/participant actions) |
| `DELETE` | `/api/games/{code}` | Archive game (requires organizerToken) |
| `POST` | `/api/email/send` | Send notification emails |
| `POST` | `/api/games/cleanup` | Cleanup expired games (cron, requires secret) |
| `GET` | `/api/config` | Frontend config (App Insights connection string) |
| `GET` | `/api/health` | Full health check with service status |
| `GET` | `/api/health/live` | Liveness probe |
| `GET` | `/api/health/ready` | Readiness probe |

## Key Patterns

### Frontend
- **Routing**: Hash-based (`#create`) + path-based (`/privacy`, `/organizer-guide`)
- **i18n**: `LanguageProvider` context + `useLanguage()` hook, 9 languages
- **Translations**: Per-language files in `src/lib/translations/` — add keys to all 9 files
- **State**: `useLocalStorage` hook for client-side persistence
- **Dark Mode**: `useDarkMode` hook + `DarkModeToggle`, toggles `.dark-theme` on `<html>`
- **Telemetry**: App Insights via `src/lib/app-insights.ts` (connection string from `/api/config`)

### API
- **Rate Limiting**: IP-based via `api/src/shared/rate-limiter.ts` (10/min create, 20/min email, 60/min default)
- **Security**: crypto-secure tokens (`randomUUID`/`randomInt`), timing-safe comparison (`safeCompare`), input limits (`INPUT_LIMITS`)
- **Error Handling**: `ApiErrorCode` enum + `createErrorResponse()` + `trackError()` pattern
- **Telemetry**: `trackEvent()` and `trackError()` via `api/src/shared/telemetry.ts`

### Creating API Functions
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/DevOpsEngineerPersona](https://github.com/Azure-Samples/DevOpsEngineerPersona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
