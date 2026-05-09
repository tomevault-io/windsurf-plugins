---
trigger: always_on
description: **Remember:** MDX documentation files are the face of your component library. They should be clear, comprehensive, and help developers quickly understand how to use components effectively.
---

**Remember:** MDX documentation files are the face of your component library. They should be clear, comprehensive, and help developers quickly understand how to use components effectively.

# Creating MDX Documentation Files

This guide explains how to create MDX documentation files for components in the Kuro UI library.

## Overview

MDX files are located in `apps/v1/content/docs/components/` and serve as the main documentation for each component. They combine Markdown with JSX to create interactive, component-rich documentation.

## File Naming Convention

MDX files should be named after the component in kebab-case:

**Examples:**
- `button.mdx` - Button component
- `checkbox-group.mdx` - Checkbox Group component
- `input-group.mdx` - Input Group component
- `radio-group.mdx` - Radio Group component

## MDX File Structure

A typical MDX file consists of the following sections:

1. **Front Matter** - Metadata about the component
2. **Component Preview** - Live demo of the component
3. **Installation** - How to install the component
4. **Usage** - Basic usage examples
5. **Examples** - Various use cases and variants
6. **API Reference** - (Optional) Props and API documentation
7. **Changelog** - (Optional) Version history and breaking changes

### Basic Template

```mdx
---
title: Component Name
description: A brief description of what the component does.
component: true
---

<ComponentPreview name="component-demo" description="A component demo" />

## Installation

<CodeTabs>

<TabsList>
  <TabsTrigger id="cli">CLI</TabsTrigger>
  <TabsTrigger id="manual">Manual</TabsTrigger>
</TabsList>
<TabsContent id="cli">

```bash
npx taki-ui@latest add component-name
```

</TabsContent>

<TabsContent id="manual">

<Steps>

<Step>Copy and paste the following code into your project.</Step>

<ComponentSource name="component-name" title="components/ui/component-name.tsx" />

<Step>Update the import paths to match your project setup.</Step>

</Steps>

</TabsContent>

</CodeTabs>

## Usage

```tsx
import { ComponentName } from "@/components/ui/component-name"
```

```tsx
<ComponentName />
```
```

## Front Matter Fields

The front matter section at the top of the MDX file contains metadata:

```yaml
---
title: Component Name
description: A brief description of what the component does.
component: true
featured: true  # Optional - shows on homepage
links:
  doc: https://external-docs-url.com
  api: https://external-api-reference-url.com
---
```

### Front Matter Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `title` | `string` | Yes | Display name of the component |
| `description` | `string` | Yes | Brief description (1-2 sentences) |
| `component` | `boolean` | Yes | Always set to `true` for component docs |
| `featured` | `boolean` | No | If `true`, shows on featured list |
| `links.doc` | `string` | No | URL to external documentation (e.g., Radix UI) |
| `links.api` | `string` | No | URL to external API reference |

### Examples:

**Simple Component:**
```yaml
---
title: Switch
description: A control that allows the user to toggle between checked and not checked.
component: true
links:
  doc: https://www.radix-ui.com/docs/primitives/components/switch
  api: https://www.radix-ui.com/docs/primitives/components/switch#api-reference
---
```

**Featured Component:**
```yaml
---
title: Button
description: Displays a button or a component that looks like a button.
featured: true
component: true
links:
  doc: https://react-spectrum.adobe.com/react-aria/Button.html
  api: https://react-spectrum.adobe.com/react-aria/Button.html#props
---
```

## Component Preview Section

The `<ComponentPreview>` component displays a live, interactive demo of your component.

### Basic Usage

```mdx
<ComponentPreview name="component-demo" description="A component demo" />
```

### Props

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `name` | `string` | Yes | Name of the demo file (without extension) |
| `description` | `string` | No | Short description shown below the demo |
| `className` | `string` | No | Custom CSS classes for styling the preview container |

### Examples:

**Basic Preview:**
```mdx
<ComponentPreview name="switch-demo" description="A switch component." />
```

**With Custom Styling:**
```mdx
<ComponentPreview
  name="field-demo"
  className="[&_.preview]:h-[800px] [&_.preview]:p-6 md:[&_.preview]:h-[850px]"
/>
```

**With Description Only:**
```mdx
<ComponentPreview
  name="radio-group-demo"
  description="A radio group with 3 items."
/>
```

### Multiple Previews

You can include multiple previews throughout the documentation:

```mdx
<ComponentPreview name="button-demo" description="A button" className="mb-4" />

```tsx showLineNumbers
<Button variant="outline">Button</Button>
<Button variant="outline" size="icon" aria-label="Submit">
  <ArrowUpIcon />
</Button>
```
```

## Installation Section

The installation section provides two methods: CLI and manual installation.

### Structure

```mdx
## Installation

<CodeTabs>

<TabsList>
  <TabsTrigger id="cli">CLI</TabsTrigger>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsartisan/taki-ui](https://github.com/jsartisan/taki-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
