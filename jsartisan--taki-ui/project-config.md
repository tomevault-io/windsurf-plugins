---
trigger: always_on
description: **Remember:** The goal of demo files is to help developers understand how to use components effectively. Focus on clarity, simplicity, and practical examples.
---

**Remember:** The goal of demo files is to help developers understand how to use components effectively. Focus on clarity, simplicity, and practical examples.

# Creating Demo Files

This guide explains how to create demo files for components in the Kuro UI library.

## Overview

Demo files showcase how to use components and are essential for documentation. The process involves three main steps:

1. Create the demo component file
2. Register it in the registry
3. Build the registry

## Step 1: Create the Demo Component File

Demo files are located in the `apps/v1/registry/new-york/examples/` directory.

### File Naming Convention

Follow this pattern: `{component-name}-{variant}.tsx`

**Examples:**
- `checkbox-demo.tsx` - Main demo
- `checkbox-disabled.tsx` - Disabled state demo
- `button-with-icon.tsx` - Button with icon demo
- `input-group-tooltip.tsx` - Input group with tooltip

### Demo File Structure

```tsx
import { ComponentName } from "@/registry/new-york/ui/component-name"

export default function ComponentDemo() {
  return (
    <div>
      <ComponentName />
    </div>
  )
}
```

### Best Practices

1. **Keep it Simple**: Each demo should illustrate one specific use case
2. **Self-contained**: Include all necessary imports and setup
3. **Export Default**: Always use default export for the demo component
4. **Descriptive Names**: Use clear, descriptive function names

### Example: Creating a Checkbox Demo

**File:** `apps/v1/registry/new-york/examples/checkbox-demo.tsx`

```tsx
import { Checkbox } from "@/registry/new-york/ui/checkbox"
import { Label } from "@/registry/new-york/ui/label"

export default function CheckboxDemo() {
  return (
    <div className="flex items-center space-x-2">
      <Checkbox id="terms" />
      <Label htmlFor="terms">Accept terms and conditions</Label>
    </div>
  )
}
```

## Step 2: Register in registry-examples.ts

After creating your demo file, register it in `apps/v1/registry/registry-examples.ts`.

### Basic Registry Entry

```typescript
{
  name: "component-demo",
  type: "registry:example",
  registryDependencies: ["component"],
  files: [
    {
      path: "examples/component-demo.tsx",
      type: "registry:example",
    },
  ],
}
```

### Registry Entry Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `string` | Yes | Unique identifier for the demo |
| `type` | `string` | Yes | Always `"registry:example"` |
| `registryDependencies` | `string[]` | Yes | List of UI components used |
| `files` | `Array<{path, type}>` | Yes | Demo file location |
| `dependencies` | `string[]` | No | External npm packages required |
| `categories` | `string[]` | No | Categories for organization |
| `meta` | `object` | No | Metadata for display configuration |
| `description` | `string` | No | Brief description of the demo |

### Example: Checkbox Registry Entry

```typescript
{
  name: "checkbox-demo",
  type: "registry:example",
  registryDependencies: ["checkbox"],
  files: [
    {
      path: "examples/checkbox-demo.tsx",
      type: "registry:example",
    },
  ],
}
```

### Example: Complex Demo with Multiple Dependencies

```typescript
{
  name: "form-rhf-demo",
  type: "registry:example",
  registryDependencies: ["field", "input", "input-group", "button", "card"],
  files: [
    {
      path: "examples/form-rhf-demo.tsx",
      type: "registry:example",
    },
  ],
  dependencies: ["react-hook-form", "@hookform/resolvers", "zod"],
}
```

### Example: Demo with Metadata

```typescript
{
  name: "calendar-hijri",
  description: "A Persian calendar.",
  type: "registry:example",
  registryDependencies: ["calendar"],
  files: [
    {
      path: "examples/calendar-hijri.tsx",
      type: "registry:example",
    },
  ],
  categories: ["calendar", "date"],
  meta: {
    iframeHeight: "600px",
    container: "w-full bg-surface min-h-svh flex px-4 py-12 items-start md:py-20 justify-center min-w-0",
    mobile: "component",
  },
}
```

### Where to Add Your Entry

Add your new registry entry to the `examples` array in `registry-examples.ts`. The array is organized alphabetically by component name, so place your entry in the appropriate location.

```typescript
export const examples: Registry["items"] = [
  {
    name: "accordion-demo",
    // ...
  },
  // ... other entries ...
  {
    name: "your-new-demo", // <-- Add here in alphabetical order
    type: "registry:example",
    registryDependencies: ["your-component"],
    files: [
      {
        path: "examples/your-new-demo.tsx",
        type: "registry:example",
      },
    ],
  },
  // ... more entries ...
]
```

## Step 3: Build the Registry

After creating the demo file and adding the registry entry, build the registry to make it available.

### Command

Navigate to the `apps/v1` directory and run:

```bash
cd apps/v1
pnpm run registry:build
```

This command:
- Validates all registry entries
- Processes demo files
- Generates registry JSON files
- Creates necessary metadata

### Expected Output

You should see output indicating successful build:

```
Building registry...
✓ Validated registry entries
✓ Processed examples
✓ Generated registry files
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsartisan/taki-ui](https://github.com/jsartisan/taki-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
