---
trigger: always_on
description: This plugin brings vulnerability intelligence into the AI coding agent development loop. Developers get automatic security scanning on commit/install, on-demand vulnerability analysis, exploit intelligence, and AI-driven remediation — all without leaving their editor.
---

# Vulnetix AI Coding Agent Plugin — Product Requirements

## Purpose

This plugin brings vulnerability intelligence into the AI coding agent development loop. Developers get automatic security scanning on commit/install, on-demand vulnerability analysis, exploit intelligence, and AI-driven remediation — all without leaving their editor.

## System Context

```mermaid
graph LR
    subgraph "Developer Machine"
        CC["Claude Code"]
        PLG["pix-ai-coding-assistant<br/>(this repo)"]
        CLI["Vulnetix CLI<br/>(../cli)"]
        CC --> PLG
        PLG --> CLI
    end

    subgraph "Vulnetix Cloud (AWS)"
        API["VDB API<br/>(../vdb-api)<br/>Vulnerability data"]
        SITE["VDB Site API<br/>(../vdb-site)<br/>Auth & accounts"]
        DB[(PostgreSQL 18<br/>VDB + Accounts)]
        API --> DB
        SITE --> DB
    end

    subgraph "Infrastructure"
        MGR["VDB Manager<br/>(../vdb-manager)<br/>Terraform IaC"]
        MGR -.->|deploys| API
        MGR -.->|deploys| SITE
        MGR -.->|provisions| DB
    end

    CLI -->|"GET /v1/vuln/*<br/>GET /v2/vuln/*"| API
    CLI -->|"POST /v1/cli/device/*<br/>GET /v1/account"| SITE

    subgraph "Third Party"
        GH["GitHub API<br/>Dependabot · CodeQL"]
        STRIPE["Stripe<br/>Subscriptions"]
    end

    PLG --> GH
    SITE --> STRIPE
```

## Repository Dependencies

### Vulnetix CLI (`../cli`)

The plugin's runtime dependency. Every skill and command shells out to CLI subcommands.

| CLI Command | Used By | VDB API Endpoint |
|-------------|---------|------------------|
| `vulnetix vdb vuln <id>` | vuln, exploits, fix, remediation skills | `GET /v1/vuln/{id}` |
| `vulnetix vdb vulns <pkg>` | vuln skill (package mode) | `GET /v1/{package}/vulns` |
| `vulnetix vdb metrics <id>` | vuln, exploits skills | `GET /v1/vuln/{id}` (metrics) |
| `vulnetix vdb exploits <id>` | exploits skill | `GET /v1/exploits/{id}` |
| `vulnetix vdb fixes <id>` | fix skill | `GET /v1/vuln/{id}/fixes` |
| `vulnetix vdb exploits-search` | exploits-search skill, vdb-exploits-search cmd | `GET /v1/exploits` |
| `vulnetix vdb remediation <id>` | remediation skill, vdb-remediation cmd | `GET /v2/vuln/{id}/remediation-plan` |
| `vulnetix auth status` | hooks (auth check) | — |
| `vulnetix env` | context enrichment | — |

The CLI handles authentication (API key via `vulnetix auth login`), request signing, and JSON response parsing.

### VDB API (`../vdb-api`)

The vulnerability data backend. Aggregates 12+ sources (CVE.org, NVD, VulnCheck, CISA KEV, GitHub/OSV, EUVD) into enriched CVE 5.1 responses.

**Key capabilities the plugin relies on:**
- Vulnerability lookup with CVSS v2/v3/v4, EPSS, KEV enrichment
- Exploit records with source attribution (ExploitDB, Metasploit, Nuclei, etc.)
- Fix intelligence with registry versions, distribution patches, source fixes
- V2 remediation plans with workarounds, advisories, CWE guidance, verification steps
- Package vulnerability listings with affected version ranges
- SBOM/manifest scanning endpoints

**API versions:** V1 (original endpoints), V2 (discrete remediation, scanning, cloud locators, malware detection)

**Authentication:** CLI exchanges org credentials at VDB Site API for JWT, then uses API key (`ApiKey {orgId}:{hexDigest}`) for VDB API requests.

### VDB Site (`../vdb-site`)

Account management backend. The plugin depends on it for:

- **Authentication:** `POST /v1/cli/device/authorize` + `/v1/cli/device/token` — RFC 8628 device grant; the user approves in a browser signed in to the Vulnetix identity provider, and the CLI receives an API key
- **Account info:** `GET /v1/account` — subscription tier, rate limits
- **Usage tracking:** `GET /v1/usage` — request counts, audit log
- **Sign-up:** browser only, at https://www.vulnetix.com/vdb-register (Community tier, free). There is no unauthenticated endpoint that mints credentials from an email.

**Subscription tiers affecting plugin behavior:**
| Tier | Requests/Day | Cost |
|------|-------------|------|
| Community | 50 | Free |
| Pro | 2,000 | $20/mo |
| Teams | 100,000 | $250/mo |
| Enterprise | Custom | Custom |

Rate limit exhaustion causes CLI commands to fail with 429 errors. The session-summary hook could surface this.

### VDB Manager (`../vdb-manager`)

Terraform IaC that provisions and deploys everything the plugin talks to:

- **ECS Fargate** — runs VDB API and Site API containers
- **PostgreSQL 18 (RDS)** — stores vulnerability data and accounts with read replicas
- **Secrets Manager** — JWT secrets, API keys, Stripe keys shared across services
- **Stripe products** — subscription tier definitions (prices, limits)
- **Lambda functions** — scheduled data processing (source ingestion, EPSS updates)
- **S3** — artifact storage for SBOMs and scan results
- **CI/CD** — GitHub Actions with AWS OIDC for deployment

Changes to rate limits, tiers, or API infrastructure are made here, not in the API repos directly.

## Plugin Feature Matrix

### Skills — LLM-Guided Workflows

| Skill | Reads Memory | Writes Memory | Uses CLI | Uses GitHub | Modifies Code |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vulnetix/pix-ai-coding-assistant](https://github.com/Vulnetix/pix-ai-coding-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
