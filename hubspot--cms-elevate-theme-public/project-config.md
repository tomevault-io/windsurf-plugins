---
trigger: always_on
description: Create module fields
---

# HubSpot Module Fields Guide

This guide helps you create properly structured `fields.tsx` files for HubSpot CMS modules. Follow these patterns and examples to ensure consistency and best practices.

## Field Library Components

Use these pre-built components from `src/components/fieldLibrary/` whenever possible:

```typescript
import {
  ButtonContent,    // Button fields with text, link, and icon options
  ButtonStyle,      // Button styling (primary/secondary/tertiary/accent)
  CardStyle,        // Card styling variants
  HeadingAndText,   // Heading text with level selection
  HeadingStyle,     // Heading style variants
  RichTextContent,  // Rich text editing with different feature sets
  SectionStyle,     // Section styling variants
  LinkStyle,        // Link styling (primary/secondary)
} from '../../fieldLibrary/index.js';
```

## Core Field Patterns

### Basic Module Structure
```typescript
import { ModuleFields, FieldGroup } from '@hubspot/cms-components/fields';

export const fields = (
  <ModuleFields>
    {/* Content fields at top level */}
    <TextField
      name="title"
      label="Title"
      default="Default Title"
    />

    {/* Style fields in STYLE tab */}
    <FieldGroup name="groupStyle" label="Styles" tab="STYLE">
      <SectionStyle sectionStyleDefault="section_variant_1" />
      <HeadingStyle headingStyleDefault="h2" />
    </FieldGroup>
  </ModuleFields>
);
```

### Repeated Content Pattern
```typescript
<RepeatedFieldGroup
  name="items"
  label="Items"
  occurrence={{
    min: 1,
    max: 4,
    default: 2
  }}
  default={[
    { title: 'Item 1' },
    { title: 'Item 2' }
  ]}
>
  <TextField
    name="title"
    label="Item Title"
    default="New Item"
  />
</RepeatedFieldGroup>
```

### Common Field Library Usage

#### HeadingAndText
```typescript
<HeadingAndText
  headingLevelDefault="h2"
  textDefault="Default Heading"
  headingTextLabel="Section Heading" // optional
/>
```

#### ButtonContent
```typescript
<ButtonContent
  textDefault="Learn More"
  linkDefault={{
    open_in_new_tab: true,
    url: { href: '#' }
  }}
  iconPositionDefault="right"
/>
```

#### RichTextContent
```typescript
<RichTextContent
  label="Description"
  featureSet="text" // or "extended"
  richTextDefault="<p>Default content</p>"
/>
```

## Available Field Types

### Basic Fields
```typescript
import {
  TextField,         // Simple text input
  RichTextField,     // Formatted text
  ImageField,        // Image uploads
  LinkField,         // URL/page links
  BooleanField,      // Toggles
  ChoiceField,       // Dropdowns/radio/checkboxes
  ColorField,        // Color selection
  NumberField,       // Numeric input
} from '@hubspot/cms-components/fields';
```

### Specialized Fields
```typescript
import {
  MenuField,          // Menu selection
  IconField,          // Icon selection
  BlogField,          // Blog selection
  LogoField,          // Logo uploads
  TextAlignmentField, // Text alignment
  AlignmentField,     // General alignment
  DateField,          // Date input
} from '@hubspot/cms-components/fields';
```

## Field Visibility

Use for conditional field display:
```typescript
<TextField
  name="conditionalField"
  label="Conditional Field"
  visibility={{
    controlling_field_path: 'someToggle',
    controlling_value_regex: 'true',
    operator: 'EQUAL'
  }}
/>
```

## Best Practices

1. **Structure**
   - Keep content fields at top level
   - Group style fields under `FieldGroup` with `tab="STYLE"`
   - Use descriptive `name` and `label` properties

2. **Defaults**
   - Always set default values
   - Use sensible defaults that work out of the box
   - Set appropriate min/max for repeated fields

3. **Required Fields**
    - Do not require fields.
    - Do not use the 'required' property.

3. **Grouping**
   - Only group related fields
   - Use consistent naming (`group` prefix for groups)
   - Keep style-related fields in STYLE tab

4. **Validation**
   - Set appropriate validation rules
   - Include help text for complex fields
   - Use visibility rules when needed

Remember: The goal is to create intuitive, well-structured fields that provide a good editing experience in the HubSpot CMS.

## Examples

### Menu Fields.tsx

```typescript
import {
  ModuleFields,
  MenuField,
  FieldGroup,
  ChoiceField,
  NumberField,
  AlignmentField,
  TextField,
} from '@hubspot/cms-components/fields';
import { SizeChoice } from '../../MenuComponent/types.js';
import LinkStyle from '../../fieldLibrary/LinkStyle/index.js';

const sizeOptions: SizeChoice[] = [
  ['none', 'None'],
  ['small', 'Small'],
  ['medium', 'Medium'],
  ['large', 'Large'],
];

export const fields = (
  <ModuleFields>
    <MenuField
      label='Menu'
      name='menu'
      default='default'
    />
    <NumberField
      label='Max menu depth'
      name='maxDepth'
      display='slider'
      min={1}
      max={3}
      helpText='Set the maximum number of menu levels to include. Must always include at least 1 level or else no menu would populate.'
      default={3}
    />
    <TextField
      label='Accessible menu name'
      name='menuName'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
