---
trigger: always_on
description: CARE is a Digital Public Good enabling TeleICU & Decentralised Administration of Healthcare Capacity across States. This is a React + TypeScript + Vite frontend application for the healthcare management system.
---

# CARE Frontend

CARE is a Digital Public Good enabling TeleICU & Decentralised Administration of Healthcare Capacity across States. This is a React + TypeScript + Vite frontend application for the healthcare management system.

## Important: Trust These Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here. This repository has custom setup requirements and workflows that must be followed exactly.

Do not run lint or prettier. For missing i18n keys, simply add the key to the end of the `public/locale/en.json` file. Do not read the i18n files, directly append items to the end of the JSON when there is an error regarding a missing key.

## Architecture Overview

### Technology Stack

- Frontend Framework: React 19.1.1 with TypeScript
- Build Tool: Vite 6.3.5 for fast development and optimized builds
- Styling: Tailwind CSS 4.1.3 with custom healthcare-specific design system
- UI Components: shadcn/ui as primary system, CAREUI for healthcare-specific components
- State Management: @tanstack/react-query for server state, React hooks for client state
- Routing: raviger for application routing
- Forms: react-hook-form with zod validation for medical data integrity
- Internationalization: i18next for multi-language healthcare interfaces
- Testing: Playwright for E2E testing of critical healthcare workflows

### Project Structure

- `src/components/` - React components organized by feature and domain
- `src/CAREUI/` - Healthcare-specific component library
- `src/pages/` - Page-level components and routing
- `src/Utils/` - Utility functions and helpers
- `src/types/` - TypeScript type definitions for medical data

## Cross-Cutting Concerns

### Accessibility

Healthcare applications must meet enhanced accessibility standards:

- WCAG 2.1 AA compliance: Required for medical applications
- Screen reader compatibility: Medical data must be accessible via assistive technology
- Keyboard navigation: Full keyboard support for clinical environments
- High contrast support: Visibility in various clinical lighting conditions
- Focus management: Clear focus indicators for complex medical workflows

### Internationalization

Multi-language support for global healthcare deployment:

- i18n Strings: All literal strings must use i18next
- Language files: English Locale files are in `public/locale/en.json`
- Locale files for Non-English languages should not be edited directly, Managed via Crowdin,
- Date/time formats: Localized formatting for medical timestamps

## Path-Specific Instructions

Specialized guidance automatically applied based on file paths:

- `careui.instructions.md` - Healthcare-specific component development
- `react-components.instructions.md` - React component architecture and patterns
- `utils.instructions.md` - Utility function standards and medical data helpers
- `typescript-types.instructions.md` - Type definitions for medical data structures
- `pages.instructions.md` - Page component architecture and routing patterns
- `hooks.instructions.md` - Custom React hooks for healthcare workflows
- `common.instructions.md` - Core utilities, permissions, and validation
- `lib.instructions.md` - Library functions
- `providers.instructions.md` - Context providers and state management
- `context.instructions.md` - React context definitions and patterns
- `config-files.instructions.md` - Build configuration and development setup

Refer to specific instruction files in `.github/instructions/` for detailed guidance on each domain.

## Coding Standards

### Code Quality

- TypeScript: Use strict TypeScript configuration for medical data safety
- ESLint: Follow configured rules for React hooks, accessibility, and code quality
- Prettier: Consistent code formatting across the healthcare application
- Component patterns: Follow established patterns in existing codebase

### Data Handling

- Type safety: Strict typing for all medical data structures and API interfaces
- Validation: Use zod schemas for runtime validation of medical data
- Error handling: Comprehensive error boundaries and user-friendly error messages
- Logging: Appropriate logging for medical workflow debugging without exposing PHI

### Documentation Requirements

- Component documentation: Include medical use cases and accessibility notes
- API documentation: Document medical data flows and validation requirements
- Accessibility notes: Document WCAG compliance and medical device compatibility
- Medical context: Explain healthcare workflows and clinical reasoning in code comments

## Working Effectively

### Environment Setup

- Node.js 24+ is required (check `.node-version` file)
- Install Node.js 24: `curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash - && sudo apt-get install -y nodejs`
- Or use nvm: `nvm install 24 && nvm use 24`

### Bootstrap, Build, and Test the Repository

- `npm install --ignore-scripts` -- installs dependencies without platform-specific binaries (takes ~16 seconds)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohcnetwork/care_fe](https://github.com/ohcnetwork/care_fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
