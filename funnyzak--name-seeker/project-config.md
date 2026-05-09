---
trigger: always_on
description: Project Architecture and File Organization Rules
---


# NameSeeker - Project Architecture Rules

## 🏗️ Project Structure

This is a cross-platform desktop application based on **Tauri 2.x + React 19 + TypeScript** for searching usernames and emails across hundreds of websites.

### Core Architecture
- **Frontend**: React 19 + TypeScript + Vite (located in `src/`)
- **Backend**: Rust + Tauri 2.x (located in `src-tauri/src/`)
- **Communication**: Tauri event system + command calls
- **Data Source**: WhatsMyName project (600+ websites)

### Key Directory Structure

```
src/                              # React Frontend
├── components/                   # UI Components
│   ├── AboutModal.tsx           # About application modal
│   ├── DisclaimerModal.tsx      # First-launch disclaimer modal
│   ├── ExportButton.tsx         # Results export functionality
│   ├── LanguageSwitcher.tsx     # Language switching component
│   ├── ProgressIndicator.tsx    # Search progress indicator
│   ├── ResultItem.tsx           # Individual result display component
│   ├── ResultsDisplay.tsx       # Results display and filtering
│   ├── ResultsFilter.tsx        # Results filtering component
│   ├── SearchForm.tsx           # Search input form with history
│   ├── SearchHistory.tsx        # Search history dropdown
│   ├── Toast.tsx                # Individual toast notification
│   ├── ToastContainer.tsx       # Toast notification container
│   └── index.ts                 # Component exports
├── hooks/                       # React Hooks
│   ├── useDisclaimer.ts         # Disclaimer state management
│   ├── useKeyboardShortcuts.ts  # Keyboard shortcuts handling
│   ├── useLanguage.ts           # Language switching and i18n management
│   ├── useSearch.ts             # Search state management
│   ├── useSearchHistory.ts      # Search history persistence
│   └── useToast.ts              # Toast notification system
├── services/                    # Service Layer
│   └── tauriApi.ts              # Tauri API wrapper with singleton pattern
├── i18n/                        # Internationalization
│   ├── index.ts                 # i18n configuration and setup
│   ├── types.ts                 # Language types and utilities
│   └── locales/                 # Translation files
│       ├── en/                  # English translations
│       │   ├── common.json      # Common UI elements
│       │   ├── search.json      # Search-related text
│       │   ├── results.json     # Results display text
│       │   ├── export.json      # Export functionality text
│       │   ├── modals.json      # Modal dialog text
│       │   ├── toast.json       # Toast notification text
│       │   └── errors.json      # Error messages
│       └── zh/                  # Chinese translations
│           ├── common.json      # Common UI elements
│           ├── search.json      # Search-related text
│           ├── results.json     # Results display text
│           ├── export.json      # Export functionality text
│           ├── modals.json      # Modal dialog text
│           ├── toast.json       # Toast notification text
│           └── errors.json      # Error messages
├── types/                       # TypeScript Definitions
│   └── index.ts                 # Complete type declarations
├── App.css                      # Application styles
├── App.tsx                      # Main application component
├── main.tsx                     # React application entry point
└── vite-env.d.ts               # Vite environment types

src-tauri/src/                    # Rust Backend
├── core/                        # Core Business Logic
│   ├── config.rs                # Application configuration and persistence
│   ├── error.rs                 # Comprehensive error handling with context
│   ├── models.rs                # Data structures and serialization schemas
│   ├── search.rs                # High-performance search engine with concurrency
│   ├── sites.rs                 # Website data management and updates
│   ├── export.rs                # Multi-format export functionality (PDF, CSV, JSON, TXT)
│   ├── utils.rs                 # HTTP utilities and metadata extraction
│   └── mod.rs                   # Module organization and public exports
├── lib.rs                       # Tauri commands, events, and application setup
└── main.rs                      # Application entry point with logging configuration

src-tauri/data/                   # Data Resources
├── wmn-data.json               # websites from WhatsMyName project
├── email-data.json              # Email search configuration and patterns
├── wmn-metadata.json            # Metadata extraction rules and patterns
└── useragents.txt              # User agent rotation for anti-detection
```

## 📋 Development Standards

### File Naming Conventions
- **React Components**: PascalCase (e.g., `SearchForm.tsx`)
- **Hooks**: camelCase starting with `use` (e.g., `useSearch.ts`)
- **Rust Modules**: snake_case (e.g., `search.rs`)
- **Type Files**: `index.ts` or `types.ts`

### Import Order
1. React related imports
2. Third-party library imports
3. Local component imports
4. Type imports
5. Style imports

### Component Organization
- One component per file
- Related components in the same directory
- Use `index.ts` for unified exports
- Separate Hook logic from UI components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [funnyzak/name-seeker](https://github.com/funnyzak/name-seeker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
