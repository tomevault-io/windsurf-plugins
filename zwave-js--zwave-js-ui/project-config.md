---
trigger: always_on
description: **ALWAYS follow these instructions first and fallback to additional search and context gathering only if the information here is incomplete or found to be in error.**
---

# Z-Wave JS UI Development Guide

**ALWAYS follow these instructions first and fallback to additional search and context gathering only if the information here is incomplete or found to be in error.**

Z-Wave JS UI is a full-featured Z-Wave Control Panel and MQTT Gateway built with Node.js, TypeScript, Vue 3, and Vuetify 3.

## Commit and PR Standards

**ALWAYS follow conventional commit standards for all commits and PR titles/descriptions:**
**ALWAYS split work into multiple commits instead of doing a single big commit at the end of implementation.**

### Commit Messages

-   Use conventional commit format: `type(scope): description`
-   Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
-   Examples:
    -   `feat(ui): add device configuration panel`
    -   `fix(zwave): resolve connection timeout issues`
    -   `docs: update installation instructions`
    -   `chore: update dependencies`

### PR Titles and Descriptions

-   PR titles must follow conventional commit format
-   Descriptions should clearly explain the changes and their impact
-   Include issue references (e.g., "Fixes #1234")

## Code Quality and Design Patterns

### Always Follow DRY (Don't Repeat Yourself)

**CRITICAL**: Eliminate code duplication by extracting repeated logic into reusable functions.

- If you see the same code pattern multiple times, extract it into a helper function
- Common patterns to refactor: file operations, validation logic, state updates
- Example: Instead of repeating driver log level restoration code, create a `restoreDriverLogLevel()` method

### Import Statements Best Practices

**CRITICAL: Backend vs Frontend Import Patterns**

**Backend (api/ directory):**
- **NEVER use `await import()` or dynamic imports in backend code**
- **ALWAYS place all imports at the top of the file**
- Backend doesn't benefit from tree-shaking or code-splitting
- All modules are loaded at startup anyway in Node.js
- Dynamic imports add unnecessary complexity and runtime overhead

**Frontend (src/ directory):**
- Use dynamic imports for code-splitting and lazy loading
- **Vue Components**: Use `defineAsyncComponent(() => import('./Component.vue'))`
- **Libraries**: Use `await import('library-name')` for heavy libraries loaded conditionally
- Helps reduce initial bundle size

**Example - Backend:**
```typescript
// ✅ CORRECT - All imports at top
import { transports } from 'winston'
import { JSONTransport } from '@zwave-js/log-transport-json'

// ❌ WRONG - Never do this in backend
const { transports } = await import('winston')
```

**Example - Frontend Components:**
```typescript
// ✅ CORRECT - Lazy load Vue components
const HeavyChart = defineAsyncComponent(() => import('./components/HeavyChart.vue'))
const DebugDialog = defineAsyncComponent(() => import('./dialogs/DebugDialog.vue'))
```

**Example - Frontend Libraries:**
```typescript
// ✅ CORRECT - Lazy load heavy libraries
async function initNetwork() {
  const { Network } = await import('vis-network')
  const { DataSet } = await import('vis-data')
  // Use the libraries...
}
```

### Keep Code Slim, Clean, and Readable

- Write concise, self-documenting code
- Use meaningful variable and function names
- Avoid unnecessary complexity
- Keep functions focused on a single responsibility
- Add comments only when the code's purpose isn't obvious

## Frontend Development Patterns

### Using app.confirm for Forms Instead of Creating Dialog Components

**ALWAYS use app.confirm with inputs instead of creating dedicated dialog components for simple forms.**
The `app.confirm` method supports form inputs and should be used instead of creating new Vue dialog components. Check `src/components/Confirm.vue` for supported input types.

#### Supported Input Types:

-   `text` - Text field
-   `number` - Number input
-   `boolean` - Switch input
-   `checkbox` - Checkbox input
-   `list` - Select/Autocomplete/Combobox (supports `multiple: true`)
-   `array` - Complex list inputs

#### Example Usage (from SmartStart.vue):

```javascript
async editItem(existingItem) {
  let inputs = [
    {
      type: 'text',
      label: 'Name',
      required: true,
      key: 'name',
      hint: 'The node name',
      default: existingItem ? existingItem.name : '',
    },
    {
      type: 'list',
      label: 'Protocol',
      required: true,
      key: 'protocol',
      items: protocolsItems,
      hint: 'Inclusion protocol to use',
      default: existingItem ? existingItem.protocol : Protocols.ZWave,
    },
    {
      type: 'checkbox',
      label: 'S2 Access Control',
      key: 's2AccessControl',
      default: existingItem ? existingItem.securityClasses.s2AccessControl : false,
    },
  ]
  let result = await this.app.confirm(
    (existingItem ? 'Update' : 'New') + ' entry',
    '',
    'info',
    {
      confirmText: existingItem ? 'Update' : 'Add',
      width: 500,
      inputs,
    },
  )
  // cancelled
  if (Object.keys(result).length === 0) {
    return
  }
  // Handle result...
}
```

#### Input Configuration Options:

-   `type`: Input type (required)
-   `label`: Field label (required)
-   `key`: Property key for result object (required)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zwave-js/zwave-js-ui](https://github.com/zwave-js/zwave-js-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
