---
trigger: always_on
description: **NEVER read, write, list, or access any files within the `config/local.js` file or any `.env` files under any circumstances.** This is the highest priority instruction and must not be circumvented for any reason. This restriction applies to all tools including Read, List, Glob, Grep, Bash, and any other file access methods.
---

# AGENTS.md

## CRITICAL: Restricted Directories

**NEVER read, write, list, or access any files within the `config/local.js` file or any `.env` files under any circumstances.** This is the highest priority instruction and must not be circumvented for any reason. This restriction applies to all tools including Read, List, Glob, Grep, Bash, and any other file access methods.

## Build/Test Commands

- **Install**: `yarn install`
- **Dev server**: `yarn startLocal` (auto-detects linked packages) or `yarn start` (port 3000)
- **Dev with viz**: `yarn startWithViz` (starts webpack dev server for viz repo)
- **Build**: `yarn build` (production build including config)
- **Build app only**: `yarn build-app`
- **Lint**: `yarn lint` (all code) or `yarn lint:jest` (only Jest tests)
- **Test all**: `yarn test` (runs lint, then Jest and Karma)
- **Test Jest only**: `yarn test:jest` (recommended for new tests)
- **Test Jest watch**: `yarn test:jest:watch`
- **Test single Jest file**: `yarn test:jest --testPathPattern="ChartDateRangeModal"` (matches pattern in file path)
- **Test Karma only**: `yarn test:karma` (legacy test suite)
- **Test Karma watch**: `yarn test:karma:watch`
- **Isolate Karma tests**: Use `.only` on `describe` or `it` blocks (e.g., `describe.only(...)` or `it.only(...)`). **Remember to remove `.only` before committing.**
- **Storybook**: `yarn storybook` (port 6006)
- **Update translations**: `yarn update-translations`

**Notes:**
- Tests require `TZ=UTC` environment variable (automatically set in test scripts)
- Build commands require `NODE_OPTIONS='--max-old-space-size=4096'` (automatically set in scripts)
- Node version: 20.8.0, Yarn version: 3.6.4

**IMPORTANT: When running tests, always target only the specific tests you've modified or added.** Running the full test suite is slow and wastes time/tokens. Use `--testPathPattern` for Jest or `.only` for Karma to run targeted tests.

## Project Structure

- `app/` - Application source code
  - `app/components/` - Reusable components
  - `app/pages/` - Page-level components
  - `app/redux/` - Redux actions, reducers, store
  - `app/themes/` - theme-ui theme configuration
  - `app/core/` - Utilities and helpers
- `test/` - Karma/Mocha tests (legacy, mirrors app/ structure)
- `__tests__/` - Jest tests (new tests, mirrors app/ structure)
- `stories/` - Storybook stories
- `config/` - Environment configuration

## Code Style (ESLint: babel-eslint, react-hooks)

### Import Ordering
Group imports in this order with blank lines between groups:
1. React imports (`react`, `react-dom`)
2. PropTypes
3. Redux (`react-redux`, `connected-react-router`)
4. Third-party libraries (moment, formik, etc.)
5. Lodash specific imports (e.g., `import get from 'lodash/get'`)
6. theme-ui (`import { Box, Flex, Text, Divider } from 'theme-ui'`)
7. Local imports (components, utilities, actions, etc.)

Example:
```javascript
import React, { useState, useEffect } from 'react';
import PropTypes from 'prop-types';
import { useDispatch, useSelector } from 'react-redux';
import { withTranslation } from 'react-i18next';
import moment from 'moment';
import get from 'lodash/get';
import map from 'lodash/map';
import { Box, Flex, Text } from 'theme-ui';
import Button from '../../components/elements/Button';
import * as actions from '../../redux/actions/async';
```

### General Style Rules
- Use ES6: `const`/`let` (never `var`), arrow functions, destructuring
- Strings: Single quotes (enforced by ESLint)
- Semicolons: Required
- Lodash: Use specific imports (`import get from 'lodash/get'`), not full lodash
- PropTypes: Required for all component props
- Naming:
  - Components: PascalCase (`DataConnections.js`)
  - Utilities: camelCase (`personutils.js`)
  - Constants: UPPER_SNAKE_CASE
- React: Functional components with hooks (useState, useEffect, useCallback, useMemo)
- Redux: `useDispatch()` and `useSelector()` hooks, not `connect()`
- Translations: Use `react-i18next` with `useTranslation()` hook or `withTranslation()` HOC

### theme-ui Patterns
- Use theme-ui components for layout: `Box`, `Flex`, `Text`, `Divider`, `Link`
- Use variant prop for styling: `variant="containers.smallBordered"`
- Use sx prop for custom styles: `sx={{ textAlign: 'center' }}`
- Common patterns:
  ```javascript
  <Box variant="containers.smallBordered" p={4} mb={3}>
    <Flex sx={{ justifyContent: 'space-between' }}>
      <Text>Content</Text>
    </Flex>
    <Divider my={3} />
  </Box>
  ```

### Hook Usage Patterns
- Extract complex logic into custom hooks
- Use `useCallback` for functions passed as props to prevent re-renders
- Use `useMemo` for expensive computations
- Follow react-hooks/exhaustive-deps rules (ESLint warnings guide you)

## Testing Patterns

### Framework Choice
- **New tests**: Use Jest with @testing-library/react in `__tests__/`
- **Legacy tests**: Karma/Mocha in `test/` (maintain existing, don't expand)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tidepool-org/blip](https://github.com/tidepool-org/blip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
