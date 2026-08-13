---
trigger: always_on
description: Expo React Native **managed project** (SDK 54) for electronic Job Safety Analysis (JSA) forms. Part of the **WellBuilt Suite** — an oilfield services platform targeting water and oil hauling companies in the Williston Basin / Bakken. Built for Liquid Gold Trucking LLC in Williston, ND as first customer. Android package name: `com.syconik801.jsaapp`.
---

# CLAUDE.md - WB JSA (Electronic Job Safety Analysis)

## Project Overview
Expo React Native **managed project** (SDK 54) for electronic Job Safety Analysis (JSA) forms. Part of the **WellBuilt Suite** — an oilfield services platform targeting water and oil hauling companies in the Williston Basin / Bakken. Built for Liquid Gold Trucking LLC in Williston, ND as first customer. Android package name: `com.syconik801.jsaapp`.

**Sister app to WB Tickets** (`C:\dev\waterticket-app`). Both share the same Firebase project (`wellbuilt-sync`) and NDIC well data.

## Tech Stack
- **Framework:** Expo SDK 54, React Native 0.81.5, React 19.1.0
- **Language:** TypeScript 5.9.2 (strict mode)
- **Navigation:** Expo Router 6.0.15 (file-based routing in `app/` folder)
- **Backend:** Firebase (Firestore) - project `wellbuilt-sync` (shared with WB Tickets)
- **Build:** EAS Build (eas.json), New Architecture enabled
- **JS Engine:** Hermes (default)
- **Well Data:** NDIC wells from Firestore (same collections as WB Tickets — `operators`, `wells`, `operatorAliases`)
- **Animations:** React Native Reanimated 4.1.1

## Project Structure
```
C:\dev\JSA\
  app/                          # Expo Router pages (file-based routing)
    (tabs)/
      index.tsx                 # HOME: Driver/truck/date, NDIC well autocomplete, job type, pusher, lease/pad
      history.tsx               # HISTORY: List of saved/completed JSAs
      _layout.tsx               # Tab navigation layout (Job Details + Saved JSAs)
    _layout.tsx                 # Root layout, theme provider, language context
    steps.tsx                   # STEPS & HAZARDS: 9 sequential JSA steps with hazards/controls
    ppe.tsx                     # PPE CHECKLIST: Personal protective equipment selection (9 standard + 4 custom)
    signoff.tsx                 # SIGN OFF: Prepared checklist, notes, signature, submit
    pdf.tsx                     # PDF GENERATION: Printable JSA report via expo-print
    completed.tsx               # Completion confirmation screen
    openJsas.tsx                # Resume/view incomplete JSAs
    viewJsa.tsx                 # View details of a completed JSA
    modal.tsx                   # Generic modal screen
    contexts/
      LanguageContext.tsx        # i18n context (English/Spanish) — WARNING: Expo Router logs a warning about missing default export, harmless
  components/
    jsa/
      InputField.tsx            # Reusable input field component
      PrimaryButton.tsx         # Reusable button component
      SummaryCard.tsx           # Card showing JSA summary
  services/
    firebase.ts                 # Firebase init + Firestore CRUD for JSAs (connected to wellbuilt-sync)
    sync.ts                     # Cloud sync logic (device ID, upload/download)
    wellData.ts                 # NDIC well data access — loads operators/wells/aliases from Firestore, search, 24hr AsyncStorage cache
  constants/
    jsaTemplate.ts              # JSA hazards, controls, PPE items, emergency contacts, Loading/Unloading step data
    colors.ts                   # Color palette (gold primary #F5A623) & spacing system
    storageKeys.ts              # AsyncStorage key constants
    theme.ts                    # Theme configuration
  hooks/
    use-color-scheme.ts         # Detect system dark/light mode
    use-theme-color.ts          # Theme color helper
  types/
    jsa.ts                      # TypeScript type definitions
  assets/
    images/
      company-logo-transparent.png  # Liquid Gold Trucking logo
```

## Key Architecture Decisions
- **Managed Expo project** (NOT bare like WB Tickets — no native modules, no android/ folder)
- **Expo Router** for file-based routing (different from WB Tickets which uses React Navigation stack)
- **Shared Firebase project:** Same `wellbuilt-sync` Firestore as WB Tickets. Reads from `operators`, `wells`, `operatorAliases` collections.
- **JSA template hardcoded:** Loading/Unloading steps, hazards, controls, PPE, and emergency contacts are in `constants/jsaTemplate.ts` for Liquid Gold. Future: configurable per company via Firestore `companies/{id}/jsaTemplate`.
- **NDIC well autocomplete:** `services/wellData.ts` loads all ~19k wells from Firestore on app start, cached 24hr in AsyncStorage. Search requires 3+ characters. Shows well name, operator, county.
- **Offline-first:** JSAs saved to AsyncStorage, synced to Firestore when connected.
- **i18n:** Custom lightweight system (no library), English/Spanish toggle, ~120+ translated strings.
- **Light theme:** White background (#F5F5F5), gold accents (#F5A623), dark text (#111111). Different from WB Tickets' dark theme.

## App Flow
1. **Home** → Enter driver name, truck #, date, job type (Loading/Unloading), pusher, well(s) via NDIC autocomplete, optional lease/pad name
2. **Steps & Hazards** → Review 9 safety steps with hazards and controls, mark each complete
3. **PPE Checklist** → Select required PPE items (9 standard + 4 customizable "Other")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tester3x/wellbuilt-jsa](https://github.com/tester3x/wellbuilt-jsa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
