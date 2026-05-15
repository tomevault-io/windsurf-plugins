---
trigger: always_on
description: When creating new Contentful components, use this template pattern based on existing components like [hero.tsx](mdc:apps/web/src/components/sections/hero.tsx), [cta.tsx](mdc:apps/web/src/components/sections/cta.tsx), and [feature-cards-with-icon.tsx](mdc:apps/web/src/components/sections/feature-cards-with-icon.tsx).
---

# Contentful Component Creation Template

## New Component Template

When creating new Contentful components, use this template pattern based on existing components like [hero.tsx](mdc:apps/web/src/components/sections/hero.tsx), [cta.tsx](mdc:apps/web/src/components/sections/cta.tsx), and [feature-cards-with-icon.tsx](mdc:apps/web/src/components/sections/feature-cards-with-icon.tsx).

### 1. Basic Component Structure Template

```typescript
"use client";

import {
  useContentfulInspectorMode,
  useContentfulLiveUpdates,
} from "@contentful/live-preview/react";
import { Badge } from "@workspace/ui/components/badge";

import type { Type[ComponentName] } from "@/lib/contentful/types";

import { ContentfulImage } from "../contentful-image";
import { ContentfulRichText } from "../contentful-richtext";
import { ContentfulButtons } from "../contentful-button";

export type [ComponentName]Props = Type[ComponentName]<"WITHOUT_UNRESOLVABLE_LINKS">;

export function [ComponentName](mdc:props: [ComponentName]Props) {
  // Apply live updates to props
  const updatedProps = useContentfulLiveUpdates(props);
  
  // Setup inspector mode for field highlighting
  const inspectorProps = useContentfulInspectorMode({
    entryId: updatedProps.sys.id,
  });

  // Destructure fields from updated props
  const { field1, field2, field3 } = updatedProps.fields ?? {};

  return (
    <section id="[section-id]" className="my-6 md:my-16">
      <div className="container mx-auto px-4 md:px-6">
        {/* Apply inspector props to each editable field */}
        <h2
          className="text-3xl font-semibold md:text-5xl"
          {...inspectorProps({ fieldId: "field1" })}
        >
          {field1}
        </h2>
        
        <ContentfulRichText
          richText={field2}
          className="text-base md:text-lg"
          {...inspectorProps({ fieldId: "field2" })}
        />
        
        {field3 && (
          <ContentfulImage
            image={field3}
            {...inspectorProps({ fieldId: "field3" })}
          />
        )}
      </div>
    </section>
  );
}
```

### 2. Component Creation Checklist

#### Required Steps:
- [ ] Add `"use client";` directive at the top
- [ ] Import `useContentfulInspectorMode` and `useContentfulLiveUpdates`
- [ ] Import TypeScript type from `@/lib/contentful/types`
- [ ] Create props type with `"WITHOUT_UNRESOLVABLE_LINKS"`
- [ ] Apply `useContentfulLiveUpdates` to props
- [ ] Setup `useContentfulInspectorMode` with `entryId`
- [ ] Destructure fields from `updatedProps.fields ?? {}`
- [ ] Apply `inspectorProps({ fieldId: "fieldName" })` to each editable element

#### Optional Contentful Components:
- [ ] `<ContentfulRichText>` for rich text fields
- [ ] `<ContentfulImage>` for image fields  
- [ ] `<ContentfulButtons>` for button arrays
- [ ] `<Badge>` for eyebrow/category fields

### 3. Field Mapping Patterns

#### Common Field Types and Components:

```typescript
// Text Fields
<h1 {...inspectorProps({ fieldId: "title" })}>
  {title}
</h1>

// Rich Text Fields
<ContentfulRichText
  richText={richText}
  className="text-base"
  {...inspectorProps({ fieldId: "richText" })}
/>

// Image Fields
<ContentfulImage
  image={image}
  width={800}
  height={600}
  className="rounded-lg"
  {...inspectorProps({ fieldId: "image" })}
/>

// Button Arrays
<ContentfulButtons
  buttons={buttons}
  className="flex gap-4"
  {...inspectorProps({ fieldId: "buttons" })}
/>

// Badge/Eyebrow Fields
{eyebrow && (
  <Badge
    variant="secondary"
    {...inspectorProps({ fieldId: "eyebrow" })}
  >
    {eyebrow}
  </Badge>
)}

// Reference Arrays (e.g., cards, items)
{cards?.map((card, index) => (
  <SubComponent
    key={`card-${card?.sys.id || index}`}
    card={card}
    {...inspectorProps({ fieldId: "cards" })}
  />
))}
```

### 4. Nested Component Pattern

For components with nested items (like cards or FAQs):

```typescript
function SubComponent({ item, ...props }: SubComponentProps) {
  const updatedItem = useContentfulLiveUpdates(item);
  const inspectorProps = useContentfulInspectorMode({
    entryId: updatedItem?.sys?.id,
  });
  
  const { title, content } = updatedItem?.fields ?? {};
  
  return (
    <div {...props}>
      <h3 {...inspectorProps({ fieldId: "title" })}>
        {title}
      </h3>
      <ContentfulRichText
        richText={content}
        {...inspectorProps({ fieldId: "content" })}
      />
    </div>
  );
}

export function MainComponent(props: MainComponentProps) {
  const updatedProps = useContentfulLiveUpdates(props);
  const inspectorProps = useContentfulInspectorMode({
    entryId: updatedProps.sys.id,
  });
  
  const { items } = updatedProps.fields ?? {};
  
  return (
    <section>
      {items?.map((item, index) => (
        <SubComponent
          key={`item-${item?.sys.id || index}`}
          item={item}
          {...inspectorProps({ fieldId: "items" })}
        />
      ))}
    </section>
  );
}
```

### 5. Naming Conventions

#### Component Names:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robotostudio/turbo-start-contentful](https://github.com/robotostudio/turbo-start-contentful) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
