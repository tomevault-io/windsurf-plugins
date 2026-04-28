---
trigger: always_on
description: - Project: leka-product-catalogs
---

# leka-product-catalogs

## Project Identity
- Project: leka-product-catalogs
- Owner: Eukrit / GO Corporation Co., Ltd.
- Notion Dashboard: https://www.notion.so/gocorp/Coding-Project-Dashboard-Claude-32c82cea8bb080f1bbd7f26770ae9e80
- GitHub Repo: https://github.com/eukrit/leka-product-catalogs
- GCP Project ID: ai-agents-go
- GCP Project Number: 538978391890
- Cloud Run Services: leka-medusa-backend (Medusa v2), leka-medusa-storefront (Next.js)
- Region: asia-southeast1
- Service Account: claude@ai-agents-go.iam.gserviceaccount.com
- Artifact Registry: asia-southeast1-docker.pkg.dev/ai-agents-go/leka-product-catalogs
- Backend: Medusa Commerce v2 (TypeScript/Node.js)
- Frontend: Next.js 15 (React/TypeScript)
- Database: Cloud SQL PostgreSQL 15
- Cache: Memorystore Redis

## What This Project Does
Multi-brand product catalog and e-commerce system powered by Medusa Commerce v2, hosted on GCP. Each brand is a Medusa Sales Channel with its own product set, pricing, and storefront. Features: product browsing, search, filtering, cart, checkout, order management, and admin dashboard. All catalogs use the **Leka Design System**.

## Related Repos
- **accounting-automation** (master) — Peak API, Xero, MCP server → `eukrit/accounting-automation`
- **business-automation** (main) — ERP gateway, shared libs, dashboard → `eukrit/business-automation`
- **areda-product-catalogs** (main) — Areda brand catalog → `eukrit/areda-product-catalogs`
- Credential files → use `Credentials Claude Code` folder + GCP Secret Manager

## MANDATORY: After every code change
1. `git add` + `git commit` + `git push origin main`
2. Cloud Build auto-deploys to Cloud Run — verify build succeeds
3. Update `eukrit/business-automation` dashboard (`docs/index.html`) if architecture changes
4. Update CHANGELOG.md with version entry

## Credentials & Secrets

### Centralized Credentials Folder
All API credentials are stored in:
```
C:\Users\eukri\OneDrive\Documents\Claude Code\Credentials Claude Code
```
Master instructions: `Credentials Claude Code/Instructions/API Access Master Instructions.txt`

### Credential Loading Rules
1. **Local development**: Load from `.env` file (gitignored) or `credentials/` folder
2. **CI/CD (Cloud Build)**: Load from GCP Secret Manager
3. **MCP connectors**: Auth handled by the MCP platform — no local credentials needed
4. **NEVER hardcode** credentials in source code or committed files
5. **NEVER commit** `.env`, `manifest.json`, `credentials/`, `*.key`, `*.pem`, token files

### GCP Secret Manager (CI/CD)
| Secret Name | Source File | Used By |
|---|---|---|
| `peak-api-token` | Peak API Credential.txt | Peak API calls |
| `xero-client-secret` | Xero Credentials.txt | Xero OAuth refresh |
| `notion-api-key` | NOTION_API_KEY.md | Notion API |
| `slack-bot-token` | Slack OAuth.txt | Slack notifications |
| `figma-token` | Figma Token.txt | Figma API |
| `n8n-webhook-key` | n8n config | n8n webhook auth |

### Credential File References
| File | Location | Purpose |
|---|---|---|
| `ai-agents-go-4c81b70995db.json` | Credentials folder | GCP service account key |
| `client_secret_538978391890-*.json` | Credentials folder | GCP OAuth client |
| `xero_tokens.json` | Credentials folder | Xero OAuth tokens (rotating) |
| `token_oauth.json` | Credentials folder | Google OAuth token |
| `token_gmail_settings.json` | Credentials folder | Gmail OAuth token |

## GCP Infrastructure
| Resource | Value |
|----------|-------|
| GCP Project | `ai-agents-go` |
| Service Account | `claude@ai-agents-go.iam.gserviceaccount.com` |
| Firestore | Native mode, database `leka-product-catalogs` (asia-southeast1) |
| GCS Bucket | `ai-agents-go-documents` (public read, uniform bucket-level access) |
| Cloud Build | GitHub connection `github-eukrit` (us-central1) |
| Cloud Run | asia-southeast1 region |

## Design System — Leka
All catalogs use the **Leka Design System** (Figma: `ER6pbDqrJ4Uo9FuldnYBfm`):

| Token | Value |
|-------|-------|
| Font | Manrope (400-800) |
| Purple | `#8003FF` |
| Navy | `#182557` |
| Cream | `#FFF9E6` |
| Magenta | `#970260` |
| Amber | `#FFA900` |
| Red-Orange | `#E54822` |
| Card radius | 16px |
| Button radius | 8px |
| Badge radius | 9999px (pill) |
| Shadow | `0px 2px 8px rgba(24,37,87,0.08)` |

## Active Brands

| Brand | Subfolder | Cloud Run URL | Products | Source |
|-------|-----------|---------------|----------|--------|
| Wisdom | `wisdom-catalog/` | TBD | 5,071 | Excel catalogs |
| Vinci Play | `vinci-catalog/` | https://catalogs.leka.studio | 1,172 | vinci-play.com scrape |

## Project Structure (enforced)
```
leka-product-catalogs/
├── CLAUDE.md                    # This file — build rules
├── CHANGELOG.md                 # Version history (all brands)
├── README.md                    # Quick start guide
├── Dockerfile                   # Root service container
├── cloudbuild.yaml              # GCP Cloud Build pipeline
├── requirements.txt             # Python dependencies
├── verify.sh                    # Post-build verification
├── manifest.example.json        # GCP config template
├── .gitignore
├── src/
│   └── main.py                  # Root health/index service
├── scripts/                     # Utility scripts
├── docs/                        # HTML deployment summaries
│   └── <brand>.html

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eukrit/leka-product-catalogs](https://github.com/eukrit/leka-product-catalogs) — distributed by [TomeVault](https://tomevault.io/claim/eukrit).
<!-- tomevault:4.0:windsurf_rules:2026-04-18 -->
