---
trigger: always_on
description: Sitecore XM Cloud-specific rules for component development and data handling
---


# Sitecore XM Cloud Starter Rules

## XM Cloud Integration

Environment Configuration:
- Always use environment variables for XM Cloud endpoints and keys
- Required variables: `SITECORE_EDGE_CONTEXT_ID`, `NEXT_PUBLIC_DEFAULT_SITE_NAME`, `NEXT_PUBLIC_SITECORE_EDGE_CONTEXT_ID`, `SITECORE_EDITING_SECRET`
- Use `.env.remote.example` as template for environment files
- Copy to `.env.local` for local development

```typescript
// Standard environment variables for XM Cloud starters
SITECORE_EDGE_CONTEXT_ID=your-context-id
NEXT_PUBLIC_DEFAULT_SITE_NAME=your-site-name  
NEXT_PUBLIC_SITECORE_EDGE_CONTEXT_ID=your-public-context-id
SITECORE_EDITING_SECRET=your-editing-secret
```

## Component Architecture Patterns

Component Structure:

- Follow Locality of Behavior pattern
- Create a tsx `ComponentName.tsx` with all varaiants of the component as well as the props
- Export named variants: `Default`, `ThreeUp`, `Slider`, `ImageBottom`, etc. from the componenet file

Prop and Interface Structure
- Extend from `ComponentProps` from `@/lib/component-props`
- Use specific interface for component parameters
- Structure fields with `data.datasource` pattern
- Include `isPageEditing` prop for variants


```typescript
import type React from 'react';
import { useSitecore } from '@sitecore-content-sdk/nextjs';
import { ComponentProps } from '@/lib/component-props';
import { Field, ImageField } from '@sitecore-jss/sitecore-jss-nextjs';

interface HeroParams {
  [key: string]: any;
}

interface HeroFields {
  title?: { jsonValue: Field<string> };
  subtitle?: { jsonValue: Field<string> };
  backgroundImage?: { jsonValue: ImageField };
}

interface HeroProps extends ComponentProps {
  params: HeroParams;
  fields: {
    data: {
      datasource: HeroFields;
    };
  };
  isPageEditing?: boolean;
}

export const Default: React.FC<HeroProps> = (props) => {
  const { page } = useSitecore();
  const { isEditing } = page.mode;
  return <HeroDefault {...props} isPageEditing={isEditing} />;
};

export const ImageBottom: React.FC<HeroProps> = (props) => {
  const { page } = useSitecore();
  const { isEditing } = page.mode;
  return <HeroImageBottom {...props} isPageEditing={isEditing} />;
};
```


## Data Source Validation

Standard Validation Pattern:
- Always check `fields?.data?.datasource` existence
- Use `NoDataFallback` component for missing datasources
- Handle both editing and preview modes
- Provide meaningful error messages

```typescript
import { NoDataFallback } from '@/utils/NoDataFallback';

export const HeroDefault: React.FC<HeroProps> = (props) => {
  const { fields, isPageEditing } = props;
  
  if (!fields?.data?.datasource) {
    return <NoDataFallback componentName="Hero" />;
  }
  
  const { title, description, image } = fields.data.datasource;
  
  // Component implementation
};
```

## Field Handling Patterns

Sitecore Field Components:
- Use `@sitecore-content-sdk/nextjs` field components
- Access field values through `jsonValue` property
- Handle optional fields with conditional rendering
- Use proper semantic HTML tags

```typescript
import { Text, RichText, Image } from '@sitecore-content-sdk/nextjs';

// Field rendering patterns
{title?.jsonValue && (
  <Text
    tag="h1"
    field={title.jsonValue}
    className="hero-title text-4xl font-bold"
  />
)}

{description?.jsonValue && (
  <RichText field={description.jsonValue} />
)}

{image?.jsonValue && (
  <Image
    field={image.jsonValue}
    alt={title?.jsonValue?.value || 'Hero image'}
    className="w-full h-auto"
  />
)}
```

## Editing Mode Handling

Page Editing Support:
- Use `useSitecore` hook to access page mode
- Check `page.mode.isEditing` for editing state
- Pass `isPageEditing` prop to variant components
- Provide different rendering for editing vs. preview

```typescript
import { useSitecore } from '@sitecore-content-sdk/nextjs';

export const Default: React.FC<ComponentProps> = (props) => {
  const { page } = useSitecore();
  const { isEditing } = page.mode;
  const isEditMode = props.isPageEditing || isEditing;
  
  if (isEditMode) {
    // Simplified rendering for editing mode
  }
  
  // Normal rendering
};
```

## Styling and UI Patterns

Tailwind and Shadcn/ui:
- Use Tailwind CSS utility classes throughout
- Import UI components from `@/components/ui/`
- Use `cn()` utility for conditional classes
- Follow container query patterns with `@container` classes

```typescript
import { cn } from '@/lib/utils';
import { Button } from '@/components/ui/button';

<div className={cn(
  '@container bg-primary rounded-default',
  'relative mx-auto my-6 max-w-7xl px-4 py-16',
  isActive && '@md:w-full'
)}>
```

## Utility Integration

Common Utilities:
- Use `NoDataFallback` for missing datasources
- Import image wrapper: `Default as ImageWrapper` from `@/components/image/ImageWrapper.dev`
- Use button components: `ButtonBase` from `@/components/button-component/ButtonComponent`
- Implement localization with `useI18n` and `dictionaryKeys`

```typescript
import { NoDataFallback } from '@/utils/NoDataFallback';
import { Default as ImageWrapper } from '@/components/image/ImageWrapper.dev';
import { ButtonBase } from '@/components/button-component/ButtonComponent';
import { useI18n } from 'next-localization';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sitecore/xmcloud-starter-js](https://github.com/Sitecore/xmcloud-starter-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
