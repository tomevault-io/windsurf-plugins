---
trigger: always_on
description: ALL `.ts`, `.tsx`, `.js`, `.jsx` files MUST start with this header BEFORE any imports:
---

# WiseMapping Frontend Rules

## 1. Copyright Header (MANDATORY)

ALL `.ts`, `.tsx`, `.js`, `.jsx` files MUST start with this header BEFORE any imports:

```
/*
 *    Copyright [2007-2025] [wisemapping]
 *
 *   Licensed under WiseMapping Public License, Version 1.0 (the "License").
 *   It is basically the Apache License, Version 2.0 (the "License") plus the
 *   "powered by wisemapping" text requirement on every single page;
 *   you may not use this file except in compliance with the License.
 *   You may obtain a copy of the license at
 *
 *       https://github.com/wisemapping/wisemapping-open-source/blob/main/LICENSE.md
 *
 *   Unless required by applicable law or agreed to in writing, software
 *   distributed under the License is distributed on an "AS IS" BASIS,
 *   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *   See the License for the specific language governing permissions and
 *   limitations under the License.
 */
```

**Exceptions**: `package.json`, `tsconfig.json`, `.md`, `.txt`, test data files, assets

## 2. Material UI Imports (CRITICAL - 500KB+ Bundle Impact)

⚠️ Pre-push hook will REJECT incorrect imports

### ✅ CORRECT (Tree-shakeable):
```typescript
// Components - one per line
import Button from '@mui/material/Button';
import Box from '@mui/material/Box';

// Icons - one per line
import SearchIcon from '@mui/icons-material/Search';
import EditIcon from '@mui/icons-material/Edit';

// Hooks from /styles
import { useTheme, styled } from '@mui/material/styles';

// Types use 'import type'
import type { PaletteMode } from '@mui/material';
import type { SvgIconProps } from '@mui/material/SvgIcon';
```

### ❌ WRONG (Bloats bundle):
```typescript
import { Button, Box } from '@mui/material';           // NO!
import { Search, Edit } from '@mui/icons-material';    // NO!
import { PaletteMode } from '@mui/material';           // NO! Use 'import type'
```

**Common Components Quick Reference**:
```typescript
// Layout: Box, Container, Grid, Stack
// Inputs: Button, TextField, Select, MenuItem, Checkbox, Switch
// Feedback: Alert, Dialog, Snackbar, CircularProgress
// Data: Table, TableBody, TableCell, TableHead, TableRow, Chip, Tooltip
// Navigation: Tabs, Tab, Drawer
// Utils: IconButton, Divider, Paper, Typography
```

**Validation**: Run `./scripts/check-mui-imports.sh` or `yarn build:analyze`

## 3. Component File Organization (MANDATORY)

Every component MUST follow the `index.tsx` pattern:

```
ComponentName/
├── index.tsx              # Main component (REQUIRED)
├── SubComponent.tsx       # Sub-components in same dir
├── helpers.ts             # Component-specific utilities
├── types.ts               # Component-specific types
└── styles.css             # Component-specific styles
```

**Rules**:
- ✅ Every component dir has `index.tsx` (main entry point)
- ✅ Sub-components stay in parent directory, named `PascalCase.tsx`
- ✅ Helper files: `helpers.ts`, `types.ts`, `constants.ts`, `utils.ts`
- ✅ Shared/reusable → `shared/` or `common/` directory
- ❌ NO components at root level without directory
- ❌ NO redundant naming: `MyComponent/MyComponent.tsx` → use `MyComponent/index.tsx`

**Examples**:
```
theme-toggle/
└── index.tsx

admin-console/
├── index.tsx
├── layout/index.tsx
├── maps-page/index.tsx
└── accounts-page/index.tsx

action-widget/pane/
├── topic-style-editor/
│   ├── index.tsx
│   ├── IconCollection.tsx
│   └── ColorPicker.tsx
└── shared/
    ├── StyledTabs.tsx
    └── StyledEditorContainer.tsx
```

**Clean Imports Result**:
```typescript
import ThemeToggle from '../common/theme-toggle';        // ✅ Clean
import MapsPage from '../admin-console/maps-page';       // ✅ Clean
// vs
import ThemeToggle from '../common/theme-toggle/ThemeToggle'; // ❌ Redundant
```

## 4. Code Standards

- **Language**: TypeScript preferred over JavaScript
- **React**: Functional components + hooks (no class components)
- **Naming**:
  - Components: `PascalCase` (`MyComponent.tsx`)
  - Utilities: `camelCase` (`myUtility.ts`)
  - Constants: `UPPER_SNAKE_CASE` (`MAX_VALUE`)
- **Git**:
  - Clear commit messages
  - No force push to `main`
  - Branch names: `feature/name`, `fix/name`

## 5. Performance

- Check for duplicate dependencies before adding packages
- Use dynamic imports for code splitting when appropriate
- Run `yarn build:analyze` to verify bundle sizes

## 6. Testing

- Unit tests: `*.test.ts` or `*.test.tsx`
- Integration tests: Cypress
- Aim for good coverage on critical paths

## 7. TypeScript Type Safety (CRITICAL)

**MANDATORY**: All TypeScript code must be properly typed. NO exceptions.

### Prohibited:
- ❌ **NEVER use `any` type** - Always use specific types
- ❌ **NEVER use `@ts-ignore` or `@ts-expect-error`** without discussion
- ❌ **NEVER leave implicit `any`** from function parameters or variables

### Required Practices:

#### 1. Import Proper Types:
```typescript
// ✅ CORRECT - Import types from mindplot
import { Topic, Designer } from '@wisemapping/mindplot';

// ❌ WRONG - Using 'any'
const topic: any = ...;
```

#### 2. Handle Nullable Types Correctly:
```typescript
// ✅ CORRECT - Properly handle Topic | null
const getTopicDepth = (topic: Topic): number => {
  let current: Topic | null = topic;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wisemapping) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
