---
trigger: always_on
description: Storybook Story Creation Guide for CMS Elevate Theme
---


# Storybook Story Creation Guide for CMS Elevate Theme

This guide teaches Claude how to create consistent, well-structured Storybook stories following the established patterns in the cms-elevate-theme project.

## Quick Start Checklist

For any new component/module story, follow this checklist:

### Essential Files (Always Required):
1. **Args File**: `[component]Args.ts` - Contains `baseArgs` with all default prop values
2. **Main Story File**: `[Component].stories.tsx` - Contains meta configuration and Default story
3. **Unified Decorator**: Import `withStorybookContainer` from shared location *(to be implemented)*

### Basic Story Structure:
```typescript
// 1. Standard imports
import type { Meta, StoryObj } from '@storybook/react';
import { ComponentName } from '../index.js';
import { baseArgs } from './componentArgs.js';
import { withStorybookContainer } from '../../../stories/sharedDecorator.js'; // (to be implemented)

// 2. Meta configuration
const meta: Meta<typeof ComponentName> = {
  title: 'Components/ComponentName', // or 'Modules/ModuleName'
  component: ComponentName,
  parameters: { layout: 'centered', docs: { description: { component: '...' } } },
  args: baseArgs,
  decorators: [withStorybookContainer()], // if needed
  tags: ['autodocs'],
};

// 3. Required exports
export default meta;
type Story = StoryObj<typeof meta>;
export const Default: Story = {};
```

### Common Additional Files:
- `[Component].styles.stories.tsx` - Style variants (when component has 3+ styles)
- `[Component].edgecases.stories.tsx` - Edge cases and error states (recommended for all)
- `[Component].sizes.stories.tsx` - Size variants (when applicable)
- `[component]Utils.ts` - Helper functions (when component has complex props)

---

## Table of Contents

1. [**File Organization Rules**](#file-organization-rules) - Directory structure, naming conventions, file types
2. [**Meta Configuration Template**](#meta-configuration-template) - Standard meta object structure, titles, parameters
3. [**Story Creation Guidelines**](#story-creation-guidelines) - Story exports, naming, structure, patterns
4. [**Utility File Patterns**](#utility-file-patterns) - Args files, utils files, unified decorator

---

## File Organization Rules

### Directory Structure

#### Components
```
src/unified-theme/components/[ComponentName]/
├── stories/
│   ├── [ComponentName].stories.tsx           # Main story file
│   ├── [ComponentName].[category].stories.tsx # Category-specific stories
│   ├── [component]Args.ts                     # Shared base arguments
│   ├── [component]Utils.ts                    # Utility functions (optional)
│   └── [component]Decorator.tsx               # Custom decorators (optional)
├── index.js
└── [ComponentName].tsx
```

#### Modules
```
src/unified-theme/components/modules/[ModuleName]/
├── stories/
│   ├── [ModuleName]Module.stories.tsx         # Main story file
│   ├── [ModuleName]Module.[category].stories.tsx # Category-specific stories
│   ├── [module]Args.ts                        # Shared base arguments
│   ├── [module]Utils.ts                       # Utility functions
│   └── [module]ContainerDecorator.tsx         # Custom decorators
├── index.js
└── [ModuleName].tsx
```

### File Naming Conventions

#### Main Story Files
- **Components**: `[ComponentName].stories.tsx`
  - Examples: `ButtonComponent.stories.tsx`, `HeadingComponent.stories.tsx`
- **Modules**: `[ModuleName]Module.stories.tsx` 
  - Examples: `MetricsModule.stories.tsx`, `CardModule.stories.tsx`

#### Category Story Files
Use the pattern `[ComponentName].[category].stories.tsx`:

**Common Categories (most components use these):**
- `.styles.stories.tsx` - Style variants (primary, secondary, etc.) - Used by most styled components
- `.edgecases.stories.tsx` - Edge cases and error states - Used by most components for robustness testing
- `.sizes.stories.tsx` - Size variants (small, medium, large) - Used when components have size options

**Frequently Used Categories:**
- `.variants.stories.tsx` - Different visual variants - Common for modules with multiple display modes
- `.orientations.stories.tsx` - Layout orientations - Used for components with horizontal/vertical layouts
- `.icons.stories.tsx` - Icon-related variations - Used for components with icon support

**Specialized Categories (use when applicable):**
- `.gaps.stories.tsx` - Spacing variations - For modules with configurable spacing
- `.counts.stories.tsx` - Different quantities/counts - For components that display variable numbers of items
- `.containers.stories.tsx` - Container/layout variations - For modules needing different container contexts
- `.content.stories.tsx` - Content variations - For components with rich content options
- `.types.stories.tsx` - Different types/modes - For components with distinct operational modes
- `.columns.stories.tsx` - Column layout variations - For grid/column-based layouts
- `.headings.stories.tsx` - Heading variations - For components with heading hierarchy options
- `.alignment.stories.tsx` - Text/content alignment - For components with alignment controls
- `.customclasses.stories.tsx` - Custom CSS class testing - For testing additional class functionality

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
