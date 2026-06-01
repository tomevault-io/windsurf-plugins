---
trigger: always_on
description: This document is designed to help AI Agents better understand and modify the Jumble project.
---

# CLAUDE.md

This document is designed to help AI Agents better understand and modify the Jumble project.

## Project Overview

Jumble is a user-friendly Nostr client for exploring relay feeds.

- **Project Name**: Jumble
- **Main Tech Stack**: React 18 + TypeScript + Vite
- **UI Framework**: Tailwind CSS + Radix UI
- **State Management**: Jotai
- **Core Protocol**: Nostr (using nostr-tools)

## Technical Architecture

### Core Dependencies

- **Build Tool**: Vite 5.x
- **Frontend Framework**: React 18.3.x + TypeScript
- **Styling Solution**:
  - Tailwind CSS (primary styling framework)
  - Radix UI (unstyled component library)
  - next-themes (theme management)
  - tailwindcss-animate (animations)
- **State Management**: Jotai 2.x
- **Routing**: path-to-regexp (custom routing solution)
- **Rich Text Editor**: TipTap 2.x
- **Nostr Protocol**: nostr-tools 2.x
- **Other Key Libraries**:
  - i18next (internationalization)
  - dayjs (date handling)
  - flexsearch (search)
  - qr-code-styling (QR codes)
  - yet-another-react-lightbox (image viewer)

### Project Structure

```
jumble/
├── src/
│   ├── components/           # React components
│   │   ├── ui/               # Base UI components (shadcn/ui style)
│   │   └── ...               # Other feature components
│   ├── providers/            # React Context Providers
│   ├── services/             # Business logic service layer
│   ├── hooks/                # Custom React Hooks
│   ├── lib/                  # Utility functions and libraries
│   ├── types/                # TypeScript type definitions
│   ├── pages/                # Page components
|   |   ├── primary           # Primary page components (Left column)
│   │   └── secondary         # secondary page components (Right column)
│   ├── layouts/              # Layout components
│   ├── i18n/                 # Internationalization resources
|   |   ├── locales           # Localization files
│   │   └── index.tx          # Basic i18n setup
│   ├── assets/               # Static assets
│   ├── App.tsx               # App root component
│   ├── PageManager.tsx       # Page manager (custom routing logic)
│   ├── routes                # Route configuration
|   |   ├── primary.tsx       # Primary routes (Left column)
│   │   └── secondary.tsx     # Secondary routes (Right column)
│   └── constants.ts          # Constants definition
├── public/                   # Public static assets
└── resources/                # Design resources
```

## Development Guide

### Environment Setup

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Lint code
npm run lint

# Format code
npm run format
```

## Code Conventions

### Component Development

1. **Component Structure**: Each major feature component is typically in its own folder, containing index.tsx and related sub-components
2. **Styling**: Use Tailwind CSS utility classes, complex components can use class-variance-authority (cva)
3. **Type Safety**: All components should have explicit TypeScript type definitions
4. **State Management**:
   - Use Jotai atoms for global state management
   - Use Context Providers for cross-component data

### Service Layer (Services)

Service files located in `src/services/` encapsulate business logic:

- `client.service.ts` - Nostr client core logic for fetching and publishing events
- `indexed-db.service.ts` - IndexedDB data storage
- `local-storage.service.ts` - LocalStorage management
- `media-upload.service.ts` - Media upload service
- `translation.service.ts` - Translation service
- `lightning.service.ts` - Lightning Network integration
- `relay-info.service.ts` - Relay information management
- `blossom.service.ts` - Blossom integration
- `custom-emoji.service.ts` - Custom emoji management
- `libre-translate.service.ts` - LibreTranslate API integration
- `media-manager.service.ts` - Managing media play state
- `modal-manager.service.ts` - Managing modal stack for back navigation (ensures modals close one by one before actual page navigation)
- `note-stats.service.ts` - Note statistics storage and retrieval (likes, zaps, reposts)
- `poll-results.service.ts` - Poll results storage and retrieval
- `post-editor-cache.service.ts` - Caching post editor content to prevent data loss
- `web.push.service.ts` - Web metadata fetching for link previews

### Providers Architecture

The app uses a multi-layered Provider nesting structure (see `App.tsx`):

```
ScreenSizeProvider
  └─ UserPreferencesProvider
      └─ ThemeProvider
          └─ ContentPolicyProvider
              └─ NostrProvider
                  └─ ... (more providers)
```

Pay attention to Provider dependencies when modifying functionality.

And some Providers are placed in `PageManager.tsx` because they need to use the `usePrimaryPage` and `useSecondaryPage` hooks.

### Routing System

- Route configuration in `src/routes/primary.tsx` and `src/routes/secondary.tsx`
- Using `PageManager.tsx` to manage page navigation, rendering, and state. Normally, you don't need to modify this file.
- Primary pages (left column) use key-based navigation
- Secondary pages (right column) use path-based navigation with stack support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CodyTseng/jumble](https://github.com/CodyTseng/jumble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
