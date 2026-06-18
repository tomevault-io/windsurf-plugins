---
trigger: always_on
description: >
---


# App Store Connect — Full Management Skill

Complete App Store Connect management via REST API and xcodebuild CLI.

## Credentials

Load credentials from `config/credentials.local.md` (gitignored). That file contains:
- Key ID, Issuer ID, Private Key path, Team ID
- Default contact info (company, email, phone, copyright)

## Authentication (Required for All API Calls)

```python
import jwt, time, requests

# Load these values from config/credentials.local.md
KEY_ID = "YOUR_KEY_ID"           # See config/credentials.local.md
ISSUER_ID = "YOUR_ISSUER_ID"    # See config/credentials.local.md
KEY_PATH = "/path/to/AuthKey.p8" # See config/credentials.local.md

with open(KEY_PATH, 'r') as f:
    private_key = f.read()

payload = {"iss": ISSUER_ID, "iat": int(time.time()), "exp": int(time.time()) + 1200, "aud": "appstoreconnect-v1"}
token = jwt.encode(payload, private_key, algorithm="ES256", headers={"kid": KEY_ID})
headers = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
```

## Core ID Resolution (Always Do First)

Most operations require these IDs. Resolve them before any update:

```python
# 1. Get APP_ID
response = requests.get("https://api.appstoreconnect.apple.com/v1/apps", headers=headers)
for app in response.json()['data']:
    print(f"{app['attributes']['name']} - {app['attributes']['bundleId']} (ID: {app['id']})")
APP_ID = "the-app-id"

# 2. Get VERSION_ID
response = requests.get(f"https://api.appstoreconnect.apple.com/v1/apps/{APP_ID}/appStoreVersions", headers=headers)
VERSION_ID = response.json()['data'][0]['id']

# 3. Get LOCALIZATION_ID (for version-level metadata)
response = requests.get(f"https://api.appstoreconnect.apple.com/v1/appStoreVersions/{VERSION_ID}/appStoreVersionLocalizations", headers=headers)
LOCALIZATION_ID = response.json()['data'][0]['id']

# 4. Get APP_INFO_ID and APP_INFO_LOC_ID (for app-level metadata)
response = requests.get(f"https://api.appstoreconnect.apple.com/v1/apps/{APP_ID}/appInfos", headers=headers)
APP_INFO_ID = response.json()['data'][0]['id']
response = requests.get(f"https://api.appstoreconnect.apple.com/v1/appInfos/{APP_INFO_ID}/appInfoLocalizations", headers=headers)
APP_INFO_LOC_ID = response.json()['data'][0]['id']
```

## Operations Index

| Area | Operations | Reference File |
|------|-----------|----------------|
| **Metadata & Localization** | Description, keywords, subtitle, copyright, category, promo text, multi-locale, AI translation | `references/metadata-and-localization.md` |
| **Screenshots & Previews** | Upload screenshots, app previews, simulator automation, screenshot sizes | `references/screenshots-and-previews.md` |
| **Customer Reviews** | List reviews, respond, delete responses, AI-assisted responses | `references/reviews-and-ratings.md` |
| **TestFlight** | Beta groups, testers, builds, beta review submission, invitations | `references/testflight.md` |
| **Submissions & Releases** | Submit for review, phased release, version increment, nominations | `references/submissions-and-releases.md` |
| **IAP & Subscriptions** | In-app purchases, subscription groups, promo offers, pricing | `references/iap-and-subscriptions.md` |
| **Reports & Analytics** | Sales reports, finance reports, download TSV data | `references/reports-and-analytics.md` |
| **Advanced Features** | In-app events, custom product pages, app clips | `references/advanced-features.md` |
| **Build & Deploy** | Xcode archive, upload, ExportOptions, one-line deploy | `references/build-and-deploy.md` |

## API Permissions

- GET apps, versions, localizations, reviews, builds, reports
- UPDATE metadata, screenshots, categories, reviews, TestFlight, releases
- CREATE versions, localizations, screenshots, IAPs, subscriptions, events

## Hard Limitations (Cannot Be Done via API)

These operations are **confirmed impossible** via the App Store Connect REST API as of April 2026.
Do NOT attempt these programmatically — they will fail or have no endpoint.

| Operation | Why Not | What to Do Instead |
|-----------|---------|-------------------|
| **Create a new app** | No `POST /v1/apps` endpoint exists. API keys get `403 FORBIDDEN`. Fastlane `produce` also cannot do this with API keys. | Create manually at https://appstoreconnect.apple.com -> Apps -> + -> New App |
| **Configure App Privacy** | No API endpoints exist for the privacy questionnaire (data collection types, tracking declarations). Apple provides only a web-based wizard. | Configure manually: ASC -> Apps -> [App] -> App Privacy -> Get Started |
| **Upload or change app icon** | Icons are embedded in the Xcode asset catalog and bundled into the binary. There is no separate icon upload endpoint. Changing an icon requires a new build. | Set icon in Xcode: Assets.xcassets -> AppIcon, then archive and upload a new build |
| **Retrieve app icon image** | Limited: `iconAssetToken` on build resources may return icon URLs, but there is no dedicated icon download endpoint. Results vary. | Best approach: extract from the Xcode project asset catalog, or use the App Store marketing artwork URL |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [199-biotechnologies/app-store-connect-skill](https://github.com/199-biotechnologies/app-store-connect-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
