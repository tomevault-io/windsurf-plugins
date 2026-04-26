---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Medication Validation Application** — a React-based QA home assignment platform that validates medication-related data (medication names, dosages, frequencies, dates of birth, and patient weight). The app includes form field detection, comprehensive validators, and a sophisticated system for detecting user testing patterns (detector/accomplishment system).

Key context: The application is designed for QA candidates to complete timed assignments with limited access windows. It monitors user interactions to detect different testing approaches and patterns.

## Build, Lint, and Test Commands

```bash
# Development server (runs on http://localhost:5173)
npm run dev

# Build for production (outputs to dist/)
npm run build

# Run linter to check code quality
npm run lint

# Run all tests
npm test

# Run a specific test file
npm test -- src/__tests__/integratedDetectorValidator.test.js

# Run tests in watch mode
npm test -- --watch
```

## Code Architecture

### Core Data Flow

1. **Access Control** (App.jsx):
   - Validates time-limited access tokens from URL parameters
   - Uses CET-based timezone handling for consistent scheduling across regions
   - Tokens encode email, start datetime, and duration (e.g., "3h")
   - Automatically redirects unencoded parameters (`?email=...&date=...&time=...&duration=...`) to encoded token format
   - Checks access validity every 5 seconds; shows "expires in X minutes" countdown timer

2. **Form Management** (ValidationForm.jsx):
   - Maintains form state for: medication, dateOfBirth, weight, dosage, frequency
   - Real-time field validation using dedicated validators
   - **Detector/Accomplishment System**: On form submission, calls detector functions for each field to identify testing patterns
   - Synced session storage with checksums to detect tampering and concurrent sessions
   - Autosaves to Google Forms every 1 minute (if changes detected)

3. **Validation & Detection Pattern**:
   - Each field has paired **detector** and **validator** modules:
     - `*Detector`: Identifies user testing patterns (e.g., boundary testing, edge cases)
     - `*Validator`: Checks if field value is valid according to rules
   - Detectors run **only on form submission**, returning an array of detection tags
   - Field-level detections feed into **Form-level detector** which detects cross-field patterns (e.g., "nominal_form" if all fields have nominal values)
   - All detections are stored in session storage as "accomplishments"

### Directory Structure

```
src/
├── App.jsx                          # Main app: access token validation, timers, header rendering
├── components/
│   ├── ValidationForm.jsx           # Form submission, field management, detector/accomplishment tracking
│   ├── Timer.jsx                    # Countdown timer showing time remaining for form access
│   └── Badges.jsx                   # UI components for displaying detected patterns
├── detectors/                       # Pattern detection functions (called on form submission)
│   ├── fieldDetector.js             # Generic field-level pattern detection
│   ├── formDetector.js              # Form-level patterns (e.g., "nominal_form", "enter_submit")
│   ├── medicationDetector.js
│   ├── dosageDetector.js
│   ├── frequencyDetector.js
│   ├── dateOfBirthDetector.js
│   └── weightDetector.js
├── validators/                      # Validation rules (return error messages or null)
│   ├── medicationValidator.js       # Validates against: placebo, aspirin, ibuprofen, paracetamol, naproxen
│   ├── dosageValidator.js
│   ├── frequencyValidator.js
│   ├── dateOfBirthValidator.js
│   └── weightValidator.js
├── hooks/                           # Custom React hooks
│   ├── useSessionStorage.js         # Basic session storage hook
│   ├── useProtectedSessionStorage.js # Session storage with checksum validation (detects tampering)
│   └── useSyncedSessionStorage.js   # Multi-window session storage sync with tampering detection
├── utils/
│   ├── accessTokenManager.js        # Token encoding/decoding, access window validation (CET timezone)
│   ├── googleFormsSubmission.js     # Submits accomplishments and form data to Google Forms
│   ├── storageChangeDetector.js     # Monitors sessionStorage for tampering and cross-window changes
│   ├── checksumUtils.js             # Checksum package creation/verification for storage integrity
│   ├── ageCalculator.js             # Age calculation from date of birth
│   └── readOnlyState.js             # deepFreeze() for immutable state
├── config/
│   ├── validationRules.js           # Centralized error messages for all validators
│   ├── badgeIcons.js                # Maps detection tags to UI icons/emojis
│   ├── badgeSortOrder.js            # Defines display order for badges
│   └── detectorCoverageData.js      # Detector coverage metadata
├── __tests__/
│   ├── integratedDetectorValidator.test.js  # Integration tests using fixtures
│   └── fixtures/                    # JSON test case files (dynamically loaded with Vite's import.meta.glob())
├── index.css                        # Global styles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zs-varga) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
