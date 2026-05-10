---
trigger: always_on
description: > **PLAN MODE**: Use Plan Mode frequently! Before implementing complex features, multi-step tasks, or making significant changes, switch to Plan Mode to think through the approach, consider edge cases, and outline the implementation strategy.
---

# AGENTS.md - AI Agent Instructions for Pumperly

> **PLAN MODE**: Use Plan Mode frequently! Before implementing complex features, multi-step tasks, or making significant changes, switch to Plan Mode to think through the approach, consider edge cases, and outline the implementation strategy.

> **IMPORTANT**: Do NOT update this file unless the user explicitly says to. Only the user can authorize changes to AGENTS.md.

> **SECURITY WARNING**: This repository is PUBLIC at [github.com/GeiserX/pumperly](https://github.com/GeiserX/pumperly). **NEVER commit secrets, API keys, passwords, tokens, or any sensitive data.** All secrets must be stored in:
> - GitHub Secrets (for CI/CD)
> - Private GitOps repositories (for docker-compose)
> - Local `.env` files (gitignored)

---

## Project Overview

**Pumperly** is an open-source, self-hostable web application that combines real-time energy price comparison with intelligent route planning — for both fuel and electric vehicles. It answers the question no other app in the world currently answers: *"What's the cheapest place to refuel or recharge along my route, and is the detour worth it?"*

- **Live URL**: https://pumperly.com
- **Repository**: https://github.com/GeiserX/pumperly
- **License**: GPL-3.0

### What Makes This Different

No product worldwide combines all three capabilities:
1. Full route planning (A to B with waypoints)
2. Real-time energy price filtering along the route (fuel + EV charging)
3. Detour time/cost calculation for every station on your route

The closest analog is **A Better Route Planner (ABRP)** for EVs — Pumperly does this for ALL vehicle types, with real-time pricing.

---

## Owner Context

**Operator**: Sergio Fernandez Rubio
**Trade Name**: GeiserCloud
**GitHub**: GeiserX

### Communication Style
- Be direct and efficient — don't over-explain
- Do the work, don't ask permission for clear tasks
- Wait for explicit deploy instruction — do NOT commit or deploy until told
- Use exact values when provided

### Preferences
- Clean, readable code without over-engineering
- Self-hosted solutions over SaaS
- Privacy-focused (cookieless analytics, minimal data collection)
- Semver versioning for Docker images (never `:latest`)
- GitOps with Portainer for infrastructure
- Docker Hub for images (`drumsergio/pumperly`)
- Tailwind CSS for styling
- TypeScript strict mode
- Do NOT add Co-Authored-By lines to commits
- Do NOT add "Generated with Claude Code" attribution anywhere

---

## Tech Stack

### Frontend
| Technology | Purpose |
|---|---|
| Next.js 15+ | App Router, Server Components |
| React 19 | UI library |
| TypeScript | Type safety (strict mode) |
| Tailwind CSS | Styling (custom components, no component library) |
| MapLibre GL JS | GPU-accelerated vector tile map rendering |
| react-map-gl | React wrapper for MapLibre (`react-map-gl/maplibre`) |
| Custom i18n | `src/lib/i18n.tsx` — React Context + inline translations, 16 locales |

### Backend
| Technology | Purpose |
|---|---|
| Next.js API Routes | REST API endpoints |
| Prisma ORM | Database access with PostGIS extensions |
| Zod | Request/response validation |

### Infrastructure
| Component | Details |
|---|---|
| PostGIS 17 | Spatial database for stations + prices (`postgis/postgis:17-3.4`) |
| Valhalla 3.5.1 | Self-hosted routing engine (`ghcr.io/gis-ops/docker-valhalla/valhalla:3.5.1`). 31-country merged PBF (~25GB) built with osmium-tool. Multiple PBFs cause SIGABRT — always merge first. Needs 24GB RAM limit for tile build. Tile count TBD after rebuild. |
| Protomaps PMTiles | Self-hosted vector map tiles on NVMe |
| OpenFreeMap | Primary tile provider (free, no API key, no rate limits) |
| Photon 1.0.1 | Geocoding / address autocomplete. Runs on `eclipse-temurin:21-jre` with official JAR. Uses OpenSearch backend (NOT old Elasticsearch). Data imported from **per-country JSONL dumps** (31 regions covering 32+ countries, ~132.7M documents). Single-pass concatenated import: all dumps downloaded in parallel, decompressed and concatenated into one file, then imported in a single `java -jar photon.jar import` invocation. |
| Caddy | Reverse proxy (existing on watchtower) |
| Docker | Multi-stage builds, images on Docker Hub |
| Portainer | Container management with GitOps |

### External Data Sources (All Free, No Auth Unless Noted)

**Government / Official APIs (9 countries):**
| Country | Source | Update Freq | Stations | Auth | Scraper Status |
|---|---|---|---|---|---|
| Spain | MITECO REST API | Daily | ~12,215 | None | ✅ Running |
| France | data.economie.gouv.fr bulk export | 10 min | ~9,868 | None | ✅ Running |
| Portugal | DGEG paginated API | Daily | ~3,186 | None (non-commercial) | ✅ Running |
| Italy | MIMIT CSV (pipe-delimited) | Daily | ~23,605 | None | ✅ Running |
| Austria | E-Control API (per-district) | Real-time | ~920 | None | ✅ Running |
| Germany | Tankerkoenig v4 API | Real-time | ~14,347 | Free API key | ✅ Running |
| UK | CMA Open Data (13 retailer JSON feeds) | Near real-time | ~3,536 | None | ✅ Running |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeiserX/Pumperly](https://github.com/GeiserX/Pumperly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
