---
trigger: always_on
description: When creating sanity schema make sure to include an appropriate icon for the schema using lucide-react or sanity icons as a fallback. Make sure it's always a named export, make sure you're always using the Sanity typescript definitions if it's a ts file.
---

# Sanity Development Guidelines

## Sanity Schema Rules

When creating sanity schema make sure to include an appropriate icon for the schema using lucide-react or sanity icons as a fallback. Make sure it's always a named export, make sure you're always using the Sanity typescript definitions if it's a ts file.

### Basic Schema Structure

For TypeScript files, always import the necessary Sanity types:

```typescript
import {defineField, defineType, defineArrayMember} from 'sanity'
```

Always use `defineField` on every field and `defineType` throughout the whole type. Only import `defineArrayMember` if needed:

```typescript
defineType({
  type: 'object',
  name: 'custom-object',
  fields: [
    defineField({
      type: 'array',
      name: 'arrayField',
      title: 'Things',
      of: [
        defineArrayMember({
          type: 'object',
          name: 'type-name-in-array',
          fields: [defineField({type: 'string', name: 'title', title: 'Title'})],
        }),
      ],
    }),
  ],
})
```

### Adding icons

When adding icons to a schema, make sure you use the default sanity/icons first, and then if no icon is relevant, refer to any other iconset the user has installed - e.g lucide-react.

### Structuring files and folders

This is a rough idea of how to structure folders and files, ensuring you always have an index within the folder to create an array of documents/blocks. Do not use these as exact names, it's used purely for layout purposes.

    │   ├── studio/
    │   │   ├── README.md
    │   │   ├── eslint.config.mjs
    │   │   ├── location.ts
    │   │   ├── package.json
    │   │   ├── prettier.config.mjs
    │   │   ├── sanity-typegen.json
    │   │   ├── sanity.cli.ts
    │   │   ├── sanity.config.ts
    │   │   ├── schema.json
    │   │   ├── structure.ts
    │   │   ├── tsconfig.json
    │   │   ├── .env.example
    │   │   ├── .gitignore
    │   │   ├── components/
    │   │   │   ├── logo.tsx
    │   │   │   └── slug-field-component.tsx
    │   │   ├── plugins/
    │   │   │   └── presentation-url.ts
    │   │   ├── schemaTypes/
    │   │   │   ├── common.ts
    │   │   │   ├── index.ts
    │   │   │   ├── blocks/
    │   │   │   │   ├── cta.ts
    │   │   │   │   ├── faq-accordion.ts
    │   │   │   │   ├── feature-cards-icon.ts
    │   │   │   │   ├── hero.ts
    │   │   │   │   ├── image-link-cards.ts
    │   │   │   │   ├── index.ts
    │   │   │   │   └── subscribe-newsletter.ts
    │   │   │   ├── definitions/
    │   │   │   │   ├── button.ts
    │   │   │   │   ├── custom-url.ts
    │   │   │   │   ├── index.ts
    │   │   │   │   ├── pagebuilder.ts
    │   │   │   │   └── rich-text.ts
    │   │   │   └── documents/
    │   │   │       ├── author.ts
    │   │   │       ├── blog.ts
    │   │   │       ├── faq.ts
    │   │   │       └── page.ts
    │   │   └── utils/
    │   │       ├── const-mock-data.ts
    │   │       ├── constant.ts
    │   │       ├── helper.ts
    │   │       ├── mock-data.ts
    │   │       ├── og-fields.ts
    │   │       ├── parse-body.ts
    │   │       ├── seo-fields.ts
    │   │       ├── slug.ts
    │   │       └── types.ts

### Layout of page builder index example

This is an example of how the blocks index file would be structured, you would create multiple of these on multiple nested routes to make it easier to create an array of files at each level, rather than bundling a large number of imports in a singular index.ts on the root

```typescript
import { callToAction } from './call-to-action';
import { exploreHero } from './explore-hero';
import { faqList } from './faq-list';
import { htmlEmbed } from './html-embed';
import { iconGrid } from './icon-grid';
import { latestDocs } from './latest-docs';
import { calculator } from './calculator';
import { navigationCards } from './navigation-cards';
import { quinstreetEmbed } from './quinstreet-embed';
import { quote } from './quote';
import { richTextBlock } from './rich-text-block';
import { socialProof } from './social-proof';
import { splitForm } from './split-form';
import { statsCard } from './stats-card';
import { trustCard } from './trust-card';
import { rvEmbed } from './rv-embed';

export const pagebuilderBlocks = [
  navigationCards,
  socialProof,
  quote,
  latestDocs,
  faqList,
  callToAction,
  trustCard,
  quinstreetEmbed,
  statsCard,
  iconGrid,
  exploreHero,
  splitForm,
  richTextBlock,
  calculator,
  htmlEmbed,
  rvEmbed,
];

export const blocks = [...pagebuilderBlocks];
```

### Common Field Templates

When writing any Sanity schema, always include a description, name, title, and type. The description should explain functionality in simple terms for non-technical users. Place description above type.

Use these templates when implementing common fields:

#### Eyebrow
```typescript
defineField({
  name: 'eyebrow',
  title: 'Eyebrow',
  description: 'The smaller text that sits above the title to provide context',
  type: 'string',
})
```

#### Title
```typescript
defineField({
  name: 'title',
  title: 'Title',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robotostudio/turbo-start-sanity](https://github.com/robotostudio/turbo-start-sanity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
