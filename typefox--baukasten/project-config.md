---
trigger: always_on
description: Apply when creating storybook comp
---


# Baukasten Storybook Guidelines

Guidelines for creating and organizing Storybook stories for Baukasten components.

## Story Structure

All component stories MUST follow this standardized structure for consistency and usability:

### 1. Interactive Story (FIRST - Always Required)

The first story must be named **"Interactive"** and expose all component properties through controls.

```tsx
export const Interactive: Story = {
  args: {
    variant: "primary",
    size: "md",
    disabled: false,
    // ... all component props with sensible defaults
  },
  parameters: {
    docs: {
      description: {
        story:
          "Interactive playground to explore all [component] properties. Try different combinations using the controls below.",
      },
    },
  },
};
```

**Purpose:**

- Playground for designers/developers to experiment
- First impression for new users
- Testing ground for QA
- Shows all available props in the Controls panel

### 2. Property Comparison Stories (Required)

Group related properties for side-by-side comparison. Common groups:

#### Variants

```tsx
export const Variants: Story = {
  render: () => (
    <div style={{ display: "flex", gap: "var(--bk-gap-sm)", flexWrap: "wrap" }}>
      <Component variant="primary">Primary</Component>
      <Component variant="secondary">Secondary</Component>
      <Component variant="ghost">Ghost</Component>
    </div>
  ),
  parameters: {
    docs: {
      description: {
        story: "Brief description of each variant and when to use them.",
      },
    },
  },
};
```

#### Sizes

```tsx
export const Sizes: Story = {
  render: () => (
    <div
      style={{ display: "flex", gap: "var(--bk-gap-sm)", alignItems: "flex-end" }}
    >
      <Component size="xs">Extra Small</Component>
      <Component size="sm">Small</Component>
      <Component size="md">Medium</Component>
      <Component size="lg">Large</Component>
      <Component size="xl">Extra Large</Component>
    </div>
  ),
  parameters: {
    docs: {
      description: {
        story:
          "Five size options available: **xs**, **sm**, **md** (default), **lg**, **xl**.",
      },
    },
  },
};
```

#### States

```tsx
export const States: Story = {
  render: () => (
    <div
      style={{
        display: "flex",
        flexDirection: "column",
        gap: "var(--bk-spacing-3)",
      }}
    >
      <div>
        <h4 style={{ marginBottom: "var(--bk-spacing-2)" }}>Default</h4>
        <Component>Default State</Component>
      </div>
      <div>
        <h4 style={{ marginBottom: "var(--bk-spacing-2)" }}>Disabled</h4>
        <Component disabled>Disabled State</Component>
      </div>
      <div>
        <h4 style={{ marginBottom: "var(--bk-spacing-2)" }}>Error</h4>
        <Component error="Error message">Error State</Component>
      </div>
    </div>
  ),
};
```

### 3. Usage Examples (Optional but Recommended)

Show practical, real-world usage patterns:

- **WithIcons** - Icon usage patterns (icon+text, icon-only)
- **WidthOptions** - Width configurations (block, wide, auto)
- **FormExamples** - Common form patterns (login, registration, validation)
- **UsageExamples** - Real-world use cases specific to the component

```tsx
export const WithIcons: Story = {
  render: () => (
    <div
      style={{
        display: "flex",
        flexDirection: "column",
        gap: "var(--bk-spacing-4)",
      }}
    >
      <div>
        <h4>Icon + Text</h4>
        <Component>
          <Icon />
          Label
        </Component>
      </div>
      <div>
        <h4>Icon Only</h4>
        <Component>
          <Icon />
        </Component>
      </div>
    </div>
  ),
};
```

### 4. Showcase Story (LAST - Always Required)

Comprehensive overview of all component capabilities. Should be the last story.

```tsx
export const Showcase: Story = {
  render: () => (
    <div
      style={{
        display: "flex",
        flexDirection: "column",
        gap: "var(--bk-spacing-6)",
        padding: "var(--bk-spacing-4)",
      }}
    >
      <div>
        <h3>Section Title</h3>
        {/* All variants/combinations */}
      </div>
      {/* More sections... */}
    </div>
  ),
  parameters: {
    layout: "fullscreen",
    docs: {
      description: {
        story:
          "Comprehensive showcase demonstrating all [component] capabilities. Use this as a reference for all available combinations.",
      },
    },
  },
};
```

## Story Meta Configuration

```tsx
const meta = {
  title: "Components/ComponentName",
  component: ComponentName,
  parameters: {
    layout: "centered", // or 'padded', 'fullscreen'
    docs: {
      description: {
        component: "Brief, clear description of the component and its purpose.",
      },
    },
  },
  tags: ["autodocs"], // REQUIRED - enables auto-generated docs
  argTypes: {
    propName: {
      control: "select", // or 'boolean', 'text', etc.
      options: ["option1", "option2"],
      description: "Clear description of what this prop does",
      table: {
        defaultValue: { summary: "default" },
      },
    },
    // ... document all props
  },
} satisfies Meta<typeof ComponentName>;
```

## Naming Conventions

### Story Names

- Use **PascalCase** for story exports
- Use descriptive, clear names
- Keep names concise but meaningful

**Good:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TypeFox/baukasten](https://github.com/TypeFox/baukasten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
