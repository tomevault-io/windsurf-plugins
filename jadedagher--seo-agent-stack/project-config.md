---
trigger: always_on
description: > **Instructions**: Replace all `{{...}}` values with your own information.
---

# {{PROJECT_NAME}} — SEO Project

## Configuration (FILL IN)

> **Instructions**: Replace all `{{...}}` values with your own information.
> Once filled, this file is the single source of truth for all agents.
> The `/setup` command can fill this automatically.

| Variable | Value |
|----------|-------|
| **Site URL** | `{{SITE_URL}}` |
| **Site Name** | `{{SITE_NAME}}` |
| **Sector** | `{{SECTOR}}` (e.g., medical imaging, e-commerce, SaaS, restaurant) |
| **Content Language** | `{{LANGUAGE}}` (e.g., fr, en, es) |
| **CSS Prefix** | `ss-` (default — SEO Stack) |

## Credentials & Connections (FILL IN)

| Resource | Identifier |
|----------|-----------|
| **Webflow Site ID** | `{{WEBFLOW_SITE_ID}}` |
| **Webflow Workspace ID** | `{{WEBFLOW_WORKSPACE_ID}}` |
| **GA4 Property ID** | `{{GA4_PROPERTY_ID}}` |
| **GSC Property** | `{{GSC_PROPERTY}}` (e.g., `sc-domain:example.com`) |

## Webflow CMS Collections (FILL IN)

> List your Webflow collections. The `/setup` command auto-discovers these.
> Agents use these IDs to publish content.

| Collection | ID | Slug | Items |
|------------|----|------|-------|
| **Articles** (blog) | `{{COLLECTION_ARTICLES_ID}}` | `/{{ARTICLES_SLUG}}` | — |
| **Service Pages** | `{{COLLECTION_SERVICES_ID}}` | `/{{SERVICES_SLUG}}` | — |
| **Sub-pages** | `{{COLLECTION_SUBPAGES_ID}}` | `/{{SUBPAGES_SLUG}}` | — |

> **Note**: You may have fewer or more collections. Add/remove rows as needed.

### CMS Schema — Articles (main blog collection)

> Adapt this schema to your actual Webflow fields. The `/setup` command auto-detects these.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | PlainText | yes | Article title |
| `slug` | PlainText | yes | URL slug (max 256 chars, alphanum) |
| `meta-description` | PlainText | yes | Max 150 characters |
| `short-info` | PlainText | yes | Short description |
| `category` | PlainText | yes | Category (free text) |
| `thumb-image` | Image | yes | Thumbnail — {{THUMB_DIMENSIONS}} px |
| `main-image` | Image | yes | Hero image — {{MAIN_IMAGE_DIMENSIONS}} px |
| `details` | RichText | yes | Article content (HTML) |

### CMS Schema — Service Pages

> Document your service/product collection fields here.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | PlainText | yes | Service name |
| `slug` | PlainText | yes | URL slug |
| `meta-title` | PlainText | yes | SEO meta title |
| `meta-description` | PlainText | yes | SEO meta description |
| `overview` | RichText | no | Main content |
| `thumb-image` | Image | no | Thumbnail |
| `main-image` | Image | no | Hero image |

## Existing Content (FILL IN)

> List pages and articles already published on your site.
> Agents use this for internal linking and to avoid cannibalization.
> The `/setup` command auto-discovers content from Webflow.

### Published Service Pages

- [Service Name 1] (`/{{SERVICES_SLUG}}/[slug]`)
- [Service Name 2] (`/{{SERVICES_SLUG}}/[slug]`)

### Published Blog Articles

- [Article Title 1] (`/{{ARTICLES_SLUG}}/[slug]`)
- [Article Title 2] (`/{{ARTICLES_SLUG}}/[slug]`)

### Static Pages

| Page | URL |
|------|-----|
| Home | `/` |
| Blog | `/{{ARTICLES_SLUG}}` |
| Contact | `/contact` |
| About | `/about` |

## URL Structure (FILL IN)

- Blog articles: `{{SITE_URL}}/{{ARTICLES_SLUG}}/[slug]`
- Service pages: `{{SITE_URL}}/{{SERVICES_SLUG}}/[slug]`
- Sub-pages: `{{SITE_URL}}/{{SUBPAGES_SLUG}}/[slug]`

## Business Context (FILL IN)

> Filled automatically by `/setup`. This context drives all SEO strategy decisions.

### Business Model

| Aspect | Details |
|--------|---------|
| **Revenue model** | `{{REVENUE_MODEL}}` (e.g., service sales, e-commerce, SaaS, lead gen) |
| **Flagship offer** | `{{FLAGSHIP_OFFER}}` |
| **Average deal size** | `{{DEAL_SIZE}}` (e.g., 50-200€, 5k-20k€) |
| **Conversion funnel** | `{{CONVERSION_FUNNEL}}` (e.g., visit → contact → call → sale) |

### Acquisition & Competition

| Aspect | Details |
|--------|---------|
| **Primary acquisition channel** | `{{PRIMARY_CHANNEL}}` (e.g., word of mouth, Google Ads, organic, social) |
| **Current organic traffic** | `{{ORGANIC_TRAFFIC_LEVEL}}` (e.g., none, ~500/mo, 10k+/mo) |
| **Main competitors** | `{{COMPETITORS}}` (URLs if available) |
| **Key differentiator** | `{{DIFFERENTIATOR}}` |

### Market & Geography

| Aspect | Details |
|--------|---------|
| **Geographic scope** | `{{GEO_SCOPE}}` (local, regional, national, international) |
| **Target areas** | `{{TARGET_AREAS}}` (cities, regions — if local/regional) |
| **Seasonal patterns** | `{{SEASONALITY}}` (e.g., summer peak, Q4 slow) |

### SEO Maturity

| Aspect | Details |
|--------|---------|
| **SEO history** | `{{SEO_HISTORY}}` (e.g., never done, tried and stopped, active agency) |
| **Publishing frequency** | `{{PUBLISH_FREQUENCY}}` (e.g., never, 1/month, 2/week) |
| **Business goals (6-12 months)** | `{{BUSINESS_GOALS}}` |

### SEO Strategy (generated by `/setup` analysis)

> This section is filled by Claude's analysis of the business context above.

| Element | Details |
|---------|---------|
| **Priority strategy** | `{{PRIORITY_STRATEGY}}` (e.g., local SEO + service pages, content marketing, technical SEO) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jadedagher/seo-agent-stack](https://github.com/jadedagher/seo-agent-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
