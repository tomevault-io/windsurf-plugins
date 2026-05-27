---
trigger: always_on
description: template_preservation:
---

# LMS MVP Project Rules
# This file defines the rules and processes for our Learning Management System project

# Template Preservation Guidelines
template_preservation:
  holy_files:
    - payload.config.ts:
        rules:
          - 'Maintain original import structure and order'
          - 'Keep original comments and placeholders'
          - 'Use defaultLexical from fields directory'
          - 'Preserve getServerSideURL utility'
          - 'No type annotations in this file'
          - 'No plugin configurations (move to plugins.ts)'
          - 'No direct editor configurations'
          - 'Minimal comments, only from original template'
          - 'Use standard Node.js imports (no node: prefix)'
          - 'Keep file-type imports compatible with Payload version'
        import_structure:
          order:
            1: 'Database adapter (mongooseAdapter)'
            2: 'Core dependencies (sharp, path, payload, url)'
            3: 'Collections'
            4: 'Globals'
            5: 'Plugins and utilities'
            6: 'Spaces module imports (last)'
        modifications:
          allowed:
            - 'Adding Spaces collections to collections array'
            - 'Adding Settings to globals array'
            - 'Adding Spaces imports at the end'
          forbidden:
            - 'Adding type imports'
            - 'Adding generateTitle/URL functions'
            - 'Modifying original template comments'
            - 'Adding new plugin configurations'
            - 'Using node: protocol imports'

    - plugins.ts:
        rules:
          - 'Keep all plugin configurations here'
          - 'Maintain original plugin order'
          - 'Keep type annotations for plugin configs'
          - 'Preserve SEO title/URL generation'
        structure:
          - 'Import all plugins first'
          - 'Import types for configurations'
          - 'Define helper functions'
          - 'Export plugins array'

  spaces_integration:
    collections_integration:
      - 'Add Spaces collections after template collections'
      - 'Maintain original collection order'
      - 'No collection configuration in payload.config.ts'

    globals_integration:
      - 'Add Settings global last in globals array'
      - 'Keep original globals order (Header, Footer)'

    imports_organization:
      - 'Group Spaces imports separately'
      - 'Place Spaces imports after template imports'
      - 'Use consistent import style'

# Technology Stack Requirements
technologies:
  frontend:
    - framework: 'Next.js@15.0.0'
    - styling: 'tailwindcss@latest'
    - ui:
        - 'shadcn-ui@latest'
        - 'aceternity-ui@latest'
        - 'magic-ui@latest'
        - 'vaul@latest'
        - 'sonner@latest'
        - 'cmdk@latest'
    - state: 'nuqs@latest'
    - forms: '@hookform/resolvers@latest'
    - validation: 'zod@latest'
    - animation:
        - 'framer-motion@latest'
        - '@legendapp/motion@latest'
        - '@formkit/auto-animate@latest'

  backend:
    - cms: 'payload@3.0.0'
    - orm: 'drizzle-orm@latest'
    - database: 'neon@latest'
    - auth:
        current: 'payload-auth@3.0.0'
        future: 'clerk@latest'
    - ai: '@vercel/ai@latest'
    - payments: 'stripe@latest'
    - uploads: 'uploadthing@latest'
    - email: 'resend@latest'
    - realtime: 'pusher@latest'

  deployment:
    - platform: 'vercel@latest'
    - monitoring:
        - 'sentry@latest'
        - '@google-cloud/logging@latest'
        - '@google-cloud/opentelemetry-cloud-monitoring-exporter@latest'
        - '@google-cloud/opentelemetry-cloud-trace-exporter@latest'
    - analytics: 'vercel-analytics@latest'

  compatibility:
    notes:
      - 'Payload CMS 3.0 requires Next.js 15 for optimal performance'
      - 'Server Actions are fully supported in Next.js 15'
      - 'App Router is the recommended approach'
      - 'React Server Components are fully utilized'
      - 'nuqs provides URL-based state management compatible with RSC'

# State Management Guidelines
state:
  principles:
    - 'Prefer server-side state management over client-side'
    - 'Use URL state for shareable and bookmarkable UI states'
    - 'Leverage React Server Components for data fetching'
    - 'Minimize client-side JavaScript'

  urlState:
    usage:
      - 'Search parameters and filters'
      - 'Pagination state'
      - 'Tab selections'
      - 'Modal states'
      - 'Sort orders'
      - 'View preferences'

    implementation:
      - 'Use nuqs for type-safe URL state management'
      - 'Define searchParams types with Zod schemas'
      - 'Implement default values for all URL parameters'
      - 'Handle URL parameter validation'

    benefits:
      - 'SEO-friendly state management'
      - 'Shareable URLs with complete state'
      - 'Reduced client-side JavaScript'
      - 'Better caching capabilities'
      - 'Native browser history support'

  serverState:
    patterns:
      - 'Use Server Components for data fetching'
      - 'Implement server actions for mutations'
      - 'Cache responses appropriately'
      - 'Handle loading and error states'

  clientState:
    limitations:
      - 'Restrict to ephemeral UI state only'
      - 'Use React.useState for temporary form state'
      - 'Avoid redundant client-side state'
      - 'Prefer URL state when possible'


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kendevco/spaces](https://github.com/kendevco/spaces) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
