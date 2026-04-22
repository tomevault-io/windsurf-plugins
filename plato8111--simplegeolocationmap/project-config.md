---
trigger: always_on
description: This file provides comprehensive guidance for developing WeWeb custom components in this repository.
---

# CLAUDE.md - WeWeb Component Development Guide

This file provides comprehensive guidance for developing WeWeb custom components in this repository.

## Development Commands

- **Install dependencies**: `npm i`
- **Serve locally**: `npm run serve --port=[PORT]` (then add custom element in WeWeb editor developer popup)
- **Build for release**: `npm run build --name=my-element` (check for build errors before release)

## WeWeb Component Architecture

This is a WeWeb custom element component built with Vue.js and configured using the WeWeb CLI framework.

### Project Structure
- `src/wwElement.vue` - Main Vue component with template, script, and scoped SCSS styling
- `ww-config.js` - WeWeb element configuration defining editor properties and settings
- `package.json` - Contains WeWeb CLI dependency and build/serve scripts

### Component Architecture
- **Props**: Component receives `content` object prop containing all configurable properties
- **Configuration**: Element properties are defined in `ww-config.js` and accessed via `props.content.propertyName`
- **Styling**: Uses scoped SCSS with Vue single-file component structure
- **WeWeb Integration**: Element integrates with WeWeb editor through configuration schema

## CRITICAL REQUIREMENTS

**THESE ARE MANDATORY AND MUST BE FOLLOWED IN ALL WeWeb COMPONENTS:**

- **MANDATORY & CRITICAL**: Use optional chaining (?.) for all content references
- **MANDATORY & CRITICAL**: All variable references, content properties, computed values, functions and any accessed data must include type safety checks (using optional chaining, nullish coalescing, type guards etc.) to prevent component crashes when values are undefined/null or of incorrect type - especially for props.content references which may not exist initially
- **MANDATORY & CRITICAL**: Every content properties must be considered reactive, when the user change their value in the editor the component must update in realtime
- **MANDATORY**: INCLUDE ALL USEFUL TRIGGERS AND INTERNAL VARIABLES
- **CRITICAL** Always import any external functions/utilities you plan to use
  - If using date-fns functions like addDays, import them: `import { addDays } from 'date-fns'`
  - If using lodash functions, import them: `import { get } from 'lodash'`
  - If using custom utilities, import them from their correct path
- Add ANY triggers that could be useful for NoCode users
- Add ANY internal variables that could be useful for NoCode users
- Think from a NoCode user perspective when adding these

## ABSOLUTELY CRITICAL TECHNICAL REQUIREMENTS

### Editor Code Blocks (MANDATORY):
- `/* wwEditor:start */` and `/* wwEditor:end */` blocks MUST be present in BOTH component code AND ww-config.js
- Required for ALL bindingValidation and propertyHelp
- EVERY wwEditor:start MUST have matching wwEditor:end
- Mismatched tags will cause catastrophic component failure

### Global Object Access (MANDATORY):
- NEVER access document/window directly
- ALWAYS use wwLib.getFrontDocument() for document
- ALWAYS use wwLib.getFrontWindow() for window
- Direct access breaks component isolation

### Component Root Element Sizing (CRITICAL):
- NEVER hardcode width/height on the root element
- Root element MUST fluidly adapt to user-defined dimensions
- Fixed dimensions on root element prevent proper NoCode customization
- Inner elements may have fixed dimensions as needed

### Array Property Requirements (ABSOLUTELY CRITICAL):
- **ALL Array properties containing objects MUST follow the WeWeb professional standard**
- **MANDATORY Array Structure in ww-config.js:**
  ```javascript
  arrayProperty: {
    label: { en: 'Items' },
    type: 'Array',
    section: 'settings',
    bindable: true,
    defaultValue: [
      { id: 'item1', name: 'Sample Item', value: 'data' }
    ],
    options: {
      expandable: true,
      getItemLabel(item) {
        return item.name || item.label || item.title || `Item ${item.id || 'Unknown'}`;
      },
      item: {
        type: 'Object',
        defaultValue: { id: 'item1', name: 'New Item', value: '' },
        options: {
          item: {
            id: { label: { en: 'ID' }, type: 'Text' },
            name: { label: { en: 'Name' }, type: 'Text' },
            value: { label: { en: 'Value' }, type: 'Text' }
          }
        }
      }
    },
    /* wwEditor:start */
    bindingValidation: {
      type: 'array',
      tooltip: 'Array of objects with required properties'
    },
    /* wwEditor:end */
  }
  ```

- **MANDATORY Formula Properties for Dynamic Field Mapping:**
  ```javascript
  // For each important field in your array items, create a formula property
  arrayPropertyIdFormula: {
    label: { en: 'ID Field' },
    type: 'Formula',
    section: 'settings',
    options: content => ({
      template: Array.isArray(content.arrayProperty) && content.arrayProperty.length > 0 ? content.arrayProperty[0] : null,
    }),
    defaultValue: {
      type: 'f',
      code: "context.mapping?.['id']",
    },
    hidden: (content, sidepanelContent, boundProps) =>
      !Array.isArray(content.arrayProperty) || !content.arrayProperty?.length || !boundProps.arrayProperty,
  },
  arrayPropertyNameFormula: {
    label: { en: 'Name Field' },
    type: 'Formula',
    section: 'settings',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/plato8111) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
