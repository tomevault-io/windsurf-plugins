---
trigger: always_on
description: Any software release (whether code, configuration, or documentation in this repository or any app utilizing this playbook) must be thoroughly audited against App Store and Google Play rejection rules.
---

# Release Review Guidelines for AI Agents

Any software release (whether code, configuration, or documentation in this repository or any app utilizing this playbook) must be thoroughly audited against App Store and Google Play rejection rules.

You MUST perform this review as if the release were about to be submitted directly to the App Store and Google Play.

## Mandatory Verification Checklist

Before certifying a release as clear to submit/ship, verify every item below. Map each to its corresponding playbook file, checklist, or script to perform the check.

### 1. Permissions
* **Verify.** Ensure no sensitive permissions (e.g., location, storage, camera) are declared without a core user-facing feature or a specific, non-generic purpose string.
* **Playbook Mapping:**
  * Checklist: `docs/PRE-SUBMISSION-CHECKLIST.md` -> "Privacy and data" & "Google Play specific"
  * Guard: `agent-os/hooks/app-store-compliance-guard.sh` (scans for standard purpose strings and sensitive permissions)
  * Rules Reference: `references/rules/privacy.md` & `references/rules/android.md`

### 2. Privacy Disclosures
* **Verify.** Ensure the app displays appropriate consent modals and nutrition/safety declarations for data collection or SDK tracking.
* **Playbook Mapping:**
  * Checklist: `docs/PRE-SUBMISSION-CHECKLIST.md` -> "Privacy and data"
  * Patterns: `data/rejection-patterns.json` -> `APPLE-5.1.2-MISSING-ATT` & `GOOGLE-DATASAFETY-MISMATCH`
  * Rules Reference: `references/rules/privacy.md`

### 3. Screenshots
* **Verify.** Check that screenshots in the store metadata show the app in actual use (not splash or login screens) and are accurate representation of the current features.
* **Playbook Mapping:**
  * Checklist: `docs/PRE-SUBMISSION-CHECKLIST.md` -> "Metadata and listing"
  * Rules Reference: `references/rules/metadata.md`

### 4. Metadata
* **Verify.** Check character limits, emojis, ALL CAPS, curse words, other platform references, ranking claims, and future feature promises.
* **Playbook Mapping:**
  * Script: `scripts/metadata-audit.py` (checks name, subtitle, description, keywords)
  * Patterns: `data/rejection-patterns.json` -> `BOTH-METADATA-DECORATION` & `APPLE-2.3-CROSS-PLATFORM-REFERENCE`
  * Rules Reference: `references/rules/metadata.md`

### 5. Age Rating
* **Verify.** Ensure that the 2026 Apple age rating questions (13+, 16+, 18+) are fully answered and that any child-directed or mature content triggers appropriate gating.
* **Playbook Mapping:**
  * Checklist: `docs/PRE-SUBMISSION-CHECKLIST.md` -> "Apple specific" (APPLE-2.3-AGE-RATING-2026)
  * Patterns: `data/rejection-patterns.json` -> `APPLE-2.3-AGE-RATING-2026`
  * Global Rules: `docs/GLOBAL-REGULATORY-2026.md`

### 6. AI Disclosures
* **Verify.** Verify that any generative AI integration has a content moderation safeguard, appropriate age rating, and, for EU users, displays an in-app notice that the user is interacting with an AI (EU AI Act Article 50(1)). Confirm third-party AI consent modals are present.
* **Playbook Mapping:**
  * Checklist: `docs/PRE-SUBMISSION-CHECKLIST.md` -> "EU specific" & "Global specific"
  * Patterns: `data/rejection-patterns.json` -> `APPLE-5.1.2-AI-NO-CONSENT-MODAL` & `BOTH-AI-GENERATED-CONTENT`
  * EU Rules: `docs/EU-REGULATORY-2026.md`

### 7. Subscription Disclosures
* **Verify.** Subscription terms, auto-renewals, billing periods, pricing hierarchy, and Terms of Use (ToS/EULA) links must be clearly laid out inside the app and in the metadata description.
* **Playbook Mapping:**
  * Checklist: `docs/PRE-SUBMISSION-CHECKLIST.md` -> "Monetization" & "Apple specific"
  * Script: `scripts/metadata-audit.py` (verifies terms/EULA links when subscriptions are mentioned)
  * Patterns: `data/rejection-patterns.json` -> `APPLE-3.1.2-MISLEADING-PRICING`

### 8. Payment Compliance
* **Verify.** Confirm that Play Billing or StoreKit is utilized for in-app digital goods (with StoreKit Restore Purchases functionality present). Ensure third-party gateways (e.g. Stripe, PayPal) are restricted to physical goods or exempt categories.
* **Playbook Mapping:**
  * Checklist: `docs/PRE-SUBMISSION-CHECKLIST.md` -> "Monetization" & "Platform mechanics gate"
  * Patterns: `data/rejection-patterns.json` -> `APPLE-3.1.1-EXTERNAL-PAYMENT`, `GOOGLE-PLAY-BILLING`, & `APPLE-RESTORE-PURCHASES-MISSING`
  * Rules Reference: `references/rules/payments.md`

### 9. Accessibility
* **Verify.** Verify accessibility support such as VoiceOver labels, Dynamic Type, contrast, and ensure compliance with accessibility standards like EN 301 549 / WCAG 2.1 AA.
* **Playbook Mapping:**
  * Checklist: `docs/PRE-SUBMISSION-CHECKLIST.md` -> "EU specific" & "Platform mechanics gate"
  * Patterns: `data/rejection-patterns.json` -> `GOOGLE-PERM-ACCESSIBILITY-MISUSE`
  * Platform Mechanics: `docs/PLATFORM-MECHANICS-2026.md`

### 10. Legal Documents
* **Verify.** Verify that required legal declarations are in place (e.g., DSA trader status, child privacy/COPPA requirements, and EU AI Act Article 4/50 compliance).
* **Playbook Mapping:**
  * EU Rules: `docs/EU-REGULATORY-2026.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mjmirza/app-store-compliance](https://github.com/mjmirza/app-store-compliance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
