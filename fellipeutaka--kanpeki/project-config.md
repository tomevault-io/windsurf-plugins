---
trigger: always_on
description: Guidelines for creating component documentation in the Kanpeki project.
---

# Component Documentation Guidelines

Guidelines for creating component documentation in the Kanpeki project.

## File Structure

Component documentation files are located in `src/content/docs/components/` and follow this pattern:

- `<component-name>.mdx` - Main documentation file
- Example components in `src/registry/examples/<component-name>/`
- Registry entries in `src/registry/examples/<component-name>/_registry.ts`

**Important**: After creating, updating, or deleting any component or file inside `src/registry/`:

1. Validate your changes:

   ```bash
   bun run registry:validate
   ```

2. If validation passes, rebuild the registry:
   ```bash
   bun run registry:build
   ```

This ensures your changes follow naming conventions before rebuilding the registry.

## MDX Frontmatter

```yaml
---
title: ComponentName
description: Brief description of what the component displays or does.
links:
  docs: https://react-aria.adobe.com/ComponentName
  api: https://react-aria.adobe.com/ComponentName#props
---
```

**Notes**:

- `title` (required): Max 32 characters
- `description` (required): Max 256 characters
- `links` (optional): Object with `docs` and/or `api` URLs

### Documentation Link Suggestions

When adding the `links.docs` field, verify the URL exists before including it. Common documentation sources:

- **React Aria Components**: `https://react-aria.adobe.com/ComponentName`
- **Base UI**: `https://base-ui.com/react/components/component-name`

You can link to any relevant documentation - these are just common options.

### Verifying Documentation Links

Before adding documentation links, verify they exist using these tools:

**React Aria Components**

- Use the `react-aria` MCP server to search component docs
- If not installed: `claude mcp add react-aria npx @react-aria/mcp@latest`
- MCP docs: https://react-aria.adobe.com/mcp

**Base UI**

- Use WebFetch with their llms.txt: `https://base-ui.com/llms.txt`
- Use the Context7 MCP server to search Base UI documentation
- Example: `mcp__context7__resolve-library-id` then `mcp__context7__get-library-docs`
- Or search the docs directly

**Any documentation**

- Use WebFetch or WebSearch tools to verify the URL exists

## Document Structure

### 1. Preview (First Section)

Start with a component preview showcasing the default/demo example:

```mdx
<ComponentPreview name="component-demo" />
```

**Important**: `ComponentPreview` only accepts a `name` prop, no `description` prop.

### 2. Installation

Use `Tabs.Root`, `Tabs.List`, `Tabs.Trigger`, and `Tabs.Content` (NOT the singular `Tabs` component):

````mdx
## Installation

<Tabs.Root>

<Tabs.List>
  <Tabs.Trigger id="cli">CLI</Tabs.Trigger>
  <Tabs.Trigger id="manual">Manual</Tabs.Trigger>
</Tabs.List>
<Tabs.Content id="cli">

```bash
npx shadcn@latest add @kanpeki/component-name
```
````

</Tabs.Content>

<Tabs.Content id="manual">

<Steps>

<Step>Copy and paste the following code into your project.</Step>

<ComponentSource name="component-name" />

<Step>Update the import paths to match your project setup.</Step>

</Steps>

</Tabs.Content>

</Tabs.Root>

````

**Key patterns**:
- Use `npm` for package manager commands (not `bun`)
- CLI command: `npx shadcn@latest add @kanpeki/component-name`
- Manual installation uses `<Steps>` component
- First step: "Copy and paste the following code into your project."
- Second step: "Update the import paths to match your project setup."

### 3. Anatomy

Use `## Anatomy` (NOT `## Usage`):

```mdx
## Anatomy

```tsx
import { Component } from "~/components/ui/component";

<Component />;
````

````

For components with multiple import patterns, use tabs:

```mdx
## Anatomy

<Tabs.Root>

<Tabs.List>
  <Tabs.Trigger id="single">Single import</Tabs.Trigger>
  <Tabs.Trigger id="multiple">Multiple imports</Tabs.Trigger>
</Tabs.List>
<Tabs.Content id="single">

```tsx
import { Component } from "~/components/ui/component";

<Component.Root>
  <Component.Item />
</Component.Root>;
````

</Tabs.Content>

<Tabs.Content id="multiple">

```tsx
import { ComponentRoot, ComponentItem } from "~/components/ui/component";

<ComponentRoot>
  <ComponentItem />
</ComponentRoot>;
```

</Tabs.Content>

</Tabs.Root>

````

### 4. Examples

**Important**: Do NOT repeat the demo example shown at the top of the page.

Each example should:
- Have a descriptive heading (e.g., `### File`, `### Disabled`, `### With Label`)
- Use `<ComponentPreview name="component-example-name" />`

```mdx
## Examples

### Variant Name

<ComponentPreview name="component-variant" />
````

## Inline Code Syntax Highlighting

When referencing components, props, or attributes in prose, use the `{:tsx}` suffix for syntax highlighting:

**Component references:**

- ✅ `<Component>{:tsx}` or `<Component.Item>{:tsx}`
- ❌ `Component` or `Component.Item`

**Props and attributes:**

- ✅ `variant{:tsx}`, `isInvalid{:tsx}`, `data-invalid{:tsx}`, `aria-label{:tsx}`
- ❌ `variant`, `isInvalid`, `data-invalid`, `aria-label`

**Prop values:**

- ✅ `variant="primary"{:tsx}`, `role="dialog"{:tsx}`
- ❌ `variant="primary"`, `role="dialog"`

Examples:

```mdx
Use `<TextField>{:tsx}` with the `isInvalid{:tsx}` prop.

Set `orientation="horizontal"{:tsx}` on `<Field.Root>{:tsx}`.

Add `aria-label{:tsx}` for accessibility.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fellipeutaka/kanpeki](https://github.com/fellipeutaka/kanpeki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
