---
trigger: always_on
description: This is the Rita project with modern TypeScript/React architecture that replaces the POC v0.01 code (now in `legacy/` folder).
---

# Rita Project - Development Documentation

This is the Rita project with modern TypeScript/React architecture that replaces the POC v0.01 code (now in `legacy/` folder).

## Project Architecture

Rita is structured as a modern microservices architecture:
- `packages/api-server/` - TypeScript API server with RabbitMQ and PostgreSQL
- `packages/client/` - **Rita Go** React/TypeScript frontend with Keycloak authentication
- `packages/mock-service/` - Mock external service for development

## Frontend Development Standards (Rita Go)

### Default Agent Usage
The Rita project uses the **fe-enterprise-agent** as the default for ALL frontend development tasks in Rita Go (`packages/client/`). The agent enforces:

- **SOC2 Type II compliance** - Security, availability, processing integrity, confidentiality, privacy
- **WCAG 2.1 AA accessibility** - Full screen reader and keyboard navigation support
- **Component-Based Architecture (CBA)** - Modular, reusable, independently deployable components
- **Platform-Driven Architecture** - Thin frontend, backend-heavy business logic
- **Server-Sent Events (SSE)** - Real-time updates via EventSource API
- **Rita Go → Actions → Rabbit → Rita Go Pattern** - Asynchronous message flow for all user actions

### Required Technical Stack
- **React 18+** with **TypeScript 5+** (strict mode)
- **TanStack Query v5** for server state management
- **Zustand** for lightweight client state
- **React Hook Form** with **Zod** validation
- **Radix UI** for accessible component primitives
- **Tailwind CSS** for utility-first styling
- **shadcn/ui** for component library foundation
- **Figma-to-shadcn** for design system implementation

### Code Standards
- All components must include proper TypeScript interfaces
- All forms must have ARIA labels and accessibility attributes
- All user inputs must be validated with Zod schemas
- All real-time features must use Server-Sent Events
- All user actions must follow audit logging requirements
- All components must be tested for accessibility compliance

**Note**: These standards apply to all developers working on this project and are enforced automatically through the fe-enterprise-agent configuration.

### Feature Flags System

Rita Go uses a **multi-scope feature flag system** for controlling features at different levels (local dev, tenant, user). This allows developers to:
- Hide incomplete features during development
- Enable beta features for specific tenants or users
- A/B test features with gradual rollouts
- Toggle debug modes and experimental features

#### Current Implementation

**Location:**
- `src/types/featureFlags.ts` - Flag registry and type definitions
- `src/lib/featureFlags.ts` - Feature flags manager (localStorage)
- `src/hooks/useFeatureFlags.ts` - React hooks for flag access
- `src/components/devtools/FeatureFlagsPanel.tsx` - Management UI
- `src/pages/DevToolsPage.tsx` - Developer tools page

**Evaluation Priority (Scope Chain):**
```
User-level (highest) → Tenant-level → Local (dev) → Default (lowest)
```
*Note: Currently only Local scope is implemented. Tenant and User scopes are architecture placeholders for future API integration.*

#### Adding a New Feature Flag

1. **Register the flag** in `src/types/featureFlags.ts`:
```typescript
export type FeatureFlagKey =
  | 'SHOW_WELCOME_MODAL'
  | 'ENABLE_DEBUG_MODE'
  | 'YOUR_NEW_FEATURE'  // Add your flag here

export const FEATURE_FLAGS: Record<FeatureFlagKey, FeatureFlagConfig> = {
  YOUR_NEW_FEATURE: {
    key: 'YOUR_NEW_FEATURE',
    label: 'Your New Feature',
    description: 'Enable the new feature you are building',
    defaultValue: false,  // Default state
    category: 'experimental',  // general | debug | experimental
  },
  // ... other flags
}
```

2. **Use the flag** in your component:
```typescript
import { useFeatureFlag } from '@/hooks/useFeatureFlags'

function MyComponent() {
  const showNewFeature = useFeatureFlag('YOUR_NEW_FEATURE')

  if (!showNewFeature) return <OldFeature />

  return <NewFeature />
}
```

#### Developer APIs

**Hook-based (Recommended):**
```typescript
// Single flag - returns boolean
const enabled = useFeatureFlag('NEW_FEATURE')

// All flags with management functions
const { flags, setFlag, toggleFlag, resetAll } = useFeatureFlags()
```

**Component-based (Future):**
```typescript
// Conditional rendering wrapper
<FeatureFlag flag="NEW_FEATURE">
  <NewFeature />
</FeatureFlag>

// With fallback
<FeatureFlag flag="NEW_FEATURE" fallback={<OldFeature />}>
  <NewFeature />
</FeatureFlag>
```

#### Developer Tools Access

Navigate to `/devtools` to:
- View all registered feature flags
- Toggle flags on/off for local development
- See flag descriptions and categories
- Reset flags to defaults

**Example:** `http://localhost:5173/devtools`

#### Testing with Feature Flags

```typescript
// Mock feature flags in tests (future utility)
import { renderWithFlags } from '@/features/feature-flags/utils/flag-testing'

describe('MyComponent', () => {
  it('shows new feature when flag enabled', () => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ehwilhelmy/AutoPilotTesting](https://github.com/ehwilhelmy/AutoPilotTesting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
