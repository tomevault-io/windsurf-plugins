---
trigger: always_on
description: >
---


## Parameters

The skill accepts an optional comma-separated list of category IDs to audit only specific checks.

| Argument | Example | Effect |
|---|---|---|
| _(none)_ | `/flutter-store-readiness` | Full audit — all 8 checks |
| Single category | `/flutter-store-readiness privacy-manifest` | Only iOS Privacy Manifest |
| Multiple categories | `/flutter-store-readiness metadata,screenshots,age-rating` | Only listed checks |

**Valid category IDs:**

| ID | What it checks |
|----|----------------|
| `privacy-manifest` | iOS PrivacyInfo.xcprivacy completeness and required reason APIs |
| `data-safety` | Android Data Safety section and privacy policy |
| `permissions` | Permission usage descriptions and justification |
| `screenshots` | Screenshot sizes, app icons, and store assets |
| `age-rating` | Age rating, content descriptors, COPPA compliance |
| `metadata` | App name, description, keywords — length limits and requirements |
| `ios` | iOS-specific: Info.plist, export compliance, encryption, entitlements |
| `android` | Android-specific: targetSdk, signing, bundle format, ProGuard |

If arguments are provided, skip all unlisted categories. The report header notes which categories were in scope.

---

# App Store Submission Readiness Skill

You are performing a structured pre-submission checklist audit on a Flutter project for
both the Apple App Store and Google Play Store. Your job is to read config files, manifests,
assets, and source code to find issues that will cause rejection or require a resubmission.

Before starting, read the detailed check patterns and requirements in:
`references/store_checks.md`

---

## Process

### Step 1 — Understand the project layout

Start with a high-level read before any searching:

- List the root directory: identify `ios/`, `android/`, `pubspec.yaml`, `assets/`
- In `ios/`: look for `Runner/Info.plist`, `Runner/PrivacyInfo.xcprivacy`, `Runner.xcodeproj/`, `Podfile`
- In `android/`: look for `app/src/main/AndroidManifest.xml`, `app/build.gradle` (or `build.gradle.kts`), `key.properties`, `proguard-rules.pro`
- Note the app's domain (fintech, health, social, kids?) — this affects severity ratings for age rating and privacy checks

### Step 2 — Run all 8 audit categories

Work through every category in order (or only the ones requested via parameters). For each:

1. **Read** the relevant files using the patterns in `references/store_checks.md`
2. **Check** each item in the category checklist
3. **Assess severity** — will this cause a **rejection** (Critical), **warning** (High), or is it a **best-practice gap** (Medium/Low)?
4. **Write a fix** — one concrete, actionable sentence

Every category produces a ✅ / ❌ / ⚠️ checklist, not just a findings list.

---

### S1 — iOS Privacy Manifest

Apple requires a `PrivacyInfo.xcprivacy` file for all apps since Spring 2024. Missing or incomplete manifests are a common rejection reason.

**Check: File exists**
- Look for `ios/Runner/PrivacyInfo.xcprivacy` (or inside an `.xcframework`)
- Flag as Critical if absent

**Check: NSPrivacyTracking**
- Must be `true` or `false` — missing key defaults to `false`
- If the app uses any advertising/analytics SDKs, `true` requires an ATT prompt

**Check: NSPrivacyTrackingDomains**
- If `NSPrivacyTracking` is `true`, list all tracking domains
- Flag if `NSPrivacyTracking: true` but `NSPrivacyTrackingDomains` is empty

**Check: NSPrivacyCollectedDataTypes**
- Must list every category of data the app collects
- Common categories: `NSPrivacyCollectedDataTypeName`, `NSPrivacyCollectedDataTypeEmailAddress`, `NSPrivacyCollectedDataTypeDeviceID`, `NSPrivacyCollectedDataTypeCrashData`, `NSPrivacyCollectedDataTypePerformanceData`
- Cross-reference with the app's dependencies (Firebase, analytics SDKs) — their data collection must be declared

**Check: NSPrivacyAccessedAPITypes — Required Reason APIs**
- Apple requires a reason for using certain APIs. Check if the app or its dependencies use any of these and declare the reason:
  - `NSPrivacyAccessedAPICategoryFileTimestamp` — file timestamp APIs
  - `NSPrivacyAccessedAPICategorySystemBootTime` — system boot time
  - `NSPrivacyAccessedAPICategoryDiskSpace` — disk space APIs
  - `NSPrivacyAccessedAPICategoryActiveKeyboards` — keyboard list
  - `NSPrivacyAccessedAPICategoryUserDefaults` — UserDefaults
- See `references/store_checks.md` for the full list of required reason codes per API category

### S2 — Android Data Safety

Google Play requires a Data Safety section in the Play Console. This is declared in the console UI (not in the APK), but the app code must be consistent with what's declared.

**Check: Privacy policy URL**
- Verify a privacy policy URL is referenced in `AndroidManifest.xml` or the app's about screen
- Flag as High if no privacy policy URL exists anywhere in the project

**Check: Data collection consistency**
- Review what the app collects (auth, analytics, crash reporting, advertising IDs) and check that the codebase is consistent with what would need to be declared
- Flag: Firebase Analytics present → "App activity" data type must be declared
- Flag: Firebase Crashlytics present → "Crash logs" must be declared
- Flag: `android.permission.READ_CONTACTS` → "Contacts" must be declared
- Flag: `android.permission.ACCESS_FINE_LOCATION` → "Precise location" must be declared

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NoaTubic/flutter-store-readiness-skill](https://github.com/NoaTubic/flutter-store-readiness-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
