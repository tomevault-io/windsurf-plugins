---
trigger: always_on
description: Alerting-specific patterns and conventions for Grafana
---


# Alerting Squad - Claude Code Configuration

This file provides context for Claude Code when working on the Grafana Alerting codebase. It contains alerting-specific patterns and references to Grafana's coding standards.

## Project Context

**Location**: `public/app/features/alerting/unified/`
**Squad**: Alerting
**Focus**: Frontend development for Grafana's unified alerting system
**Tech Stack**: React, TypeScript, Redux Toolkit, RTK Query, Emotion, Jest, React Testing Library, MSW

## Grafana Coding Standards

**IMPORTANT**: Always follow Grafana's official style guides. Do not duplicate standards here - reference the source files:

### Required Reading

1. **Frontend Style Guide**: [../../../../../contribute/style-guides/frontend.md](../../../../../contribute/style-guides/frontend.md)
   - Naming conventions, component patterns, TypeScript, exports
   - Function declarations for components, callback props with "on" prefix

2. **Testing Guidelines**: [../../../../../contribute/style-guides/testing.md](../../../../../contribute/style-guides/testing.md)
   - React Testing Library, query priorities, user event setup

3. **Styling Guide**: [../../../../../contribute/style-guides/styling.md](../../../../../contribute/style-guides/styling.md)
   - Emotion usage, `useStyles2` hook patterns

4. **Redux Framework**: [../../../../../contribute/style-guides/redux.md](../../../../../contribute/style-guides/redux.md)
   - Redux Toolkit patterns, reducer testing

5. **Alerting Testing Guide**: [./TESTING.md](./TESTING.md)
   - MSW API mocking, permission mocking, data source setup

### Alerting-Specific Conventions

**Use @grafana/alerting Package**:

- **Always check @grafana/alerting for shared components/hooks** before creating new ones:

  ```typescript
  // Good - Use exported components/hooks from @grafana/alerting
  import { AlertLabel, alertingMatchers } from '@grafana/alerting';

  // Check what's available before reimplementing
  ```

**Layout Components**:

- **Prefer @grafana/ui layout components** over styled divs:

  ```typescript
  // Good - Use Box for simple layout/spacing
  import { Box } from '@grafana/ui';
  <Box marginLeft={1}>Content</Box>

  // Good - Use Stack for flex layouts
  import { Stack } from '@grafana/ui';
  <Stack direction="column" gap={2}>
    <div>Item 1</div>
    <div>Item 2</div>
  </Stack>

  // Avoid - Custom styled divs when layout components exist
  <div className={styles.wrapper}>Content</div>
  ```

## Alerting Codebase Structure

### Key Directories

- `api/` - RTK Query API slices for data fetching
- `components/` - Feature-specific React components organized by domain
- `hooks/` - Reusable custom hooks for logic and data fetching
- `rule-editor/` - Alert rule creation and editing forms
- `rule-list/` - Alert rules list views (v1 and v2)
- `state/` - Redux state management and context providers
- `utils/` - Utility functions and helpers
- `types/` - TypeScript type definitions
- `mocks/` - MSW mock server setup for testing
- `testSetup/` - Test utilities and configuration

### Component Domains

- `alert-groups/` - Alert grouping and filtering
- `contact-points/` - Contact point configuration
- `notification-policies/` - Notification routing policies
- `mute-timings/` - Mute timing windows
- `silences/` - Silence management
- `receivers/` - Receiver configuration
- `templates/` - Notification templates
- `permissions/` - Permission management
- `settings/` - Alertmanager settings

## State Management Patterns

### RTK Query (Primary - Preferred)

**IMPORTANT**: Our direction is to use RTK Query for data fetching, NOT Redux. Do not create new RTKQ endpoints, those should be created manually.

- API slices in `api/` directory
- Custom base query in `api/alertingApi.ts`
- Automatic caching with 2-minute polling: `RULE_LIST_POLL_INTERVAL_MS`
- Key APIs: `alertRuleApi`, `alertmanagerApi`, `prometheusApi`, `receiversApi`

**For new features**: Always use RTK Query hooks for data fetching:

```typescript
import { useGetAlertRulesQuery } from '../api/alertRuleApi';

const { data, isLoading, error } = useGetAlertRulesQuery(params);
```

### Redux Toolkit (Legacy)

**Avoid for new features** - Use RTK Query instead

- Legacy reducers exist in `state/reducers/`
- Use state selectors: `useUnifiedAlertingSelector`
- Only modify if maintaining existing Redux code

### Context Providers

- `AlertmanagerContext` - Alertmanager selection state, for managing Alertmanager entities for a specific Alertmanager data source.
- `SettingsContext` - Settings state – used in `public/app/features/alerting/unified/components/settings`
- `WorkbenchContext` - Workbench state used in the alert triage feature `public/app/features/alerting/unified/triage`

### Forms

- Use `react-hook-form` (v7) for all forms
- See `rule-editor/alert-rule-form/` for patterns

## Alerting-Specific Testing Patterns

See [./TESTING.md](./TESTING.md) for comprehensive testing guide. Key points:

### API Mocking with MSW

**REQUIRED**: Use MSW for all API mocking (not `jest.fn()`) – though it's fine to use this function for unit testing.

```typescript
import { mockApi } from '../mockApi';

// Mock common endpoints
mockApi.eval(); // for AlertingQueryRunner
// If helper doesn't exist, add it to mockApi.ts
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GridProtectionAlliance/openHistorian](https://github.com/GridProtectionAlliance/openHistorian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
