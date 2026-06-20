---
trigger: always_on
description: Hivebrite Admin API CLI — users, companies, events, groups, donations, memberships, emailings, mentoring, news, projects, media center, forums, and more. OAuth2 auth (password grant or refresh token) or bearer token.
---


# Hivebrite by @altf1be

Full Hivebrite Admin API CLI covering users, companies, events, groups, donations, memberships, emailings, mentoring, news, projects, media center, forums, admins, approvals, roles, receipts, categories, comments, posts, audit logs, engagement scoring, payment accounts, network settings, and more.

## Setup

1. Obtain API credentials from your Hivebrite admin panel (Settings > Integrations or API).
2. Set environment variables (or create `.env` in `{baseDir}`):

```
# Required — your Hivebrite instance URL
HIVEBRITE_BASE_URL=https://yourcommunity.hivebrite.com

# Auth Option 1: Bearer token (simplest)
HIVEBRITE_ACCESS_TOKEN=your-access-token

# Auth Option 2: OAuth2 password grant (set all four):
# HIVEBRITE_CLIENT_ID=your-client-id
# HIVEBRITE_CLIENT_SECRET=your-client-secret
# HIVEBRITE_ADMIN_EMAIL=admin@example.com
# HIVEBRITE_ADMIN_PASSWORD=your-password

# Auth Option 3: OAuth2 refresh token grant (set all three):
# HIVEBRITE_CLIENT_ID=your-client-id
# HIVEBRITE_CLIENT_SECRET=your-client-secret
# HIVEBRITE_REFRESH_TOKEN=your-refresh-token

# Optional
# HIVEBRITE_MAX_RESULTS=25
```

3. Install dependencies: `cd {baseDir} && npm install`

## API Info

- **Base URL:** `{HIVEBRITE_BASE_URL}/api/admin/v2/` (some endpoints use v1 or v3)
- **Auth:** OAuth2 (`grant_type=password` or `grant_type=refresh_token`) or Bearer token
- **Pagination:** RFC-5988 Link headers. Params: `page` (default 1), `per_page` (default 25, max 100)
- **Rate limits:** 300 requests/minute. Max 15 errors (5xx) per minute before throttling.

## Commands

### Me

```bash
node {baseDir}/scripts/hivebrite.mjs me
```

### Settings

```bash
node {baseDir}/scripts/hivebrite.mjs settings customizable-attributes
node {baseDir}/scripts/hivebrite.mjs settings fields-of-study
node {baseDir}/scripts/hivebrite.mjs settings industries
node {baseDir}/scripts/hivebrite.mjs settings job-functions
node {baseDir}/scripts/hivebrite.mjs settings currencies
```

### Network

```bash
node {baseDir}/scripts/hivebrite.mjs network info
node {baseDir}/scripts/hivebrite.mjs network sub-networks
node {baseDir}/scripts/hivebrite.mjs network citizenships
```

### Users

```bash
# List / search users
node {baseDir}/scripts/hivebrite.mjs users list
node {baseDir}/scripts/hivebrite.mjs users list --query "john"

# CRUD
node {baseDir}/scripts/hivebrite.mjs users read --id 123
node {baseDir}/scripts/hivebrite.mjs users create --email "jane@example.com" --firstname Jane --lastname Doe
node {baseDir}/scripts/hivebrite.mjs users update --id 123 --phone "+1234567890"
node {baseDir}/scripts/hivebrite.mjs users delete --id 123 --confirm

# Sub-resources
node {baseDir}/scripts/hivebrite.mjs users experiences --user-id 123
node {baseDir}/scripts/hivebrite.mjs users educations --user-id 123
node {baseDir}/scripts/hivebrite.mjs users notification-settings --user-id 123
node {baseDir}/scripts/hivebrite.mjs users postal-addresses --user-id 123
node {baseDir}/scripts/hivebrite.mjs users group-membership --user-id 123

# Special actions
node {baseDir}/scripts/hivebrite.mjs users find-by-field --field email --value "jane@example.com"
node {baseDir}/scripts/hivebrite.mjs users notify --id 123 --subject "Hello" --message "Welcome!"
node {baseDir}/scripts/hivebrite.mjs users activate --id 123
```

### Experiences (standalone)

```bash
node {baseDir}/scripts/hivebrite.mjs experiences list
node {baseDir}/scripts/hivebrite.mjs experiences read --id 456
node {baseDir}/scripts/hivebrite.mjs experiences create --user-id 123 --title "Engineer" --organization "Acme"
node {baseDir}/scripts/hivebrite.mjs experiences update --id 456 --title "Senior Engineer"
node {baseDir}/scripts/hivebrite.mjs experiences delete --id 456 --confirm
node {baseDir}/scripts/hivebrite.mjs experiences customizable-attributes
```

### Educations (standalone)

```bash
node {baseDir}/scripts/hivebrite.mjs educations list
node {baseDir}/scripts/hivebrite.mjs educations read --id 789
node {baseDir}/scripts/hivebrite.mjs educations create --user-id 123 --degree "MSc" --school "MIT"
node {baseDir}/scripts/hivebrite.mjs educations update --id 789 --field "Computer Science"
node {baseDir}/scripts/hivebrite.mjs educations delete --id 789 --confirm
node {baseDir}/scripts/hivebrite.mjs educations customizable-attributes
```

### Emailings

```bash
# Categories
node {baseDir}/scripts/hivebrite.mjs emailings categories list
node {baseDir}/scripts/hivebrite.mjs emailings categories read --id 10
node {baseDir}/scripts/hivebrite.mjs emailings categories create --name "Newsletter"
node {baseDir}/scripts/hivebrite.mjs emailings categories update --id 10 --name "Monthly Newsletter"
node {baseDir}/scripts/hivebrite.mjs emailings categories delete --id 10 --confirm

# Campaigns
node {baseDir}/scripts/hivebrite.mjs emailings campaigns list
node {baseDir}/scripts/hivebrite.mjs emailings campaigns read --id 20
node {baseDir}/scripts/hivebrite.mjs emailings campaigns create --subject "Spring Update" --name "Spring 2026"
node {baseDir}/scripts/hivebrite.mjs emailings campaigns update --id 20 --subject "Updated Subject"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ALT-F1-OpenClaw/openclaw-skill-hivebrite-by-altf1be](https://github.com/ALT-F1-OpenClaw/openclaw-skill-hivebrite-by-altf1be) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
