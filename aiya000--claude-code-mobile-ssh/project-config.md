---
trigger: always_on
description: This file provides guidance on how Claude Code (claude.ai/code) handles the code in this repository.
---

# CLAUDE.md

This file provides guidance on how Claude Code (claude.ai/code) handles the code in this repository.

## Conversations

Use Japanese for conversations with developer.

<!--
For non-Japanese speakers:

Sorry for this instruction.
This project's main author is a Japanese speaker.

Tell 'use Englishe (or another language)' to Claude Code if you want to use another language manually.
Or tell me how to switch the language setting of Claude Code :)
-->

## Writing CLAUDE.md

Use English in this file.

Add a blank line between the title and the first list item if adding new sections with list items.
Also do not add trailing dot to list items like `- An item.`
For example:

```markdown
## Section Title

- First item
- Second item
```

## Project Overview

See README.md for about of Project Overview.
his app is mobile-first experience with PWA functionality.

## Development Commands

Use **bun** as package manager and development server.

```shell-session
# Start development server
$ bun dev

# Run typecheck
$ bun typecheck

# Run lint
$ bun lint # do prettier processing than eslint
$ bun prettier # Format the code with prettier
$ bun eslint # do Lint with the next lint (strict mode)

# Run format
$ bun fix # format the code with prettier and prettier
$ bun fix # fix problems with both prettier and eslint
$ bun fix:prettier # fix only prettier problems
$ bun fix:eslint # fix only eslint problems

# Build production build
$ bun run build # NOTE! `bun build` is a different command. Don't use it for this usage
```

Open [http://localhost:3000](http://localhost:3000) to view the application.

```shell-session
# If WSL
$ wslview 'http://localhost:3000'
```

## Architecture

### Technology Stack

- **Next.js** and App Router
- **TypeScript** and strict type safety enforcement
- **React** and **vanilla-extract** for UI components and styling
- **Bun** as package manager and runtime
- **PWA** functionality for mobile experience
- **SSG** to upload gh-pages. MUST NOT USE SSR and any servers

### Project Structure

```
src/
├─ app/ # Next.js app router page
├─ models/ # Generic types and generic functions for those types
├─ utils/ # Generic functions for TypeScript's built-in types, or functions to types or types to some types that exist in models, etc., but are not preferred to be included in models
├─ components/ # React components (atomic design)
     ├─ atoms/ # Basic UI elements
     ├─ molecules/ # Composition component
     ├─ organism/ # complex components
     ├─ templates/ # page-level components (only templates/AtPageName for one app/somepage.tsx)
```

#### models/

- Files under this directory will only contain the type that represents a single subject and a general function
  - It does not contain multiple subjects/context
  - It may contain more than one type, but only one subject/context

---

NOTE:
The following example uses Zod, but the project does not yet use Zod.
But this is not the point, so please reread it accordingly.
Also, if this project already uses Zod, and this sentence still exists, simply delete this sentence.

---

OK: Yen.ts
-- The type Yen is the subject.

```typescript
import { z } from 'zod'

const coreSchema = z.number().int().nonnegative()

export const yenSchema = coreSchema.branded('Yen')
export type Yen = z.infer<typeof yenSchema>

// Although number is not the subject,
// since it is a universal property that Yen can be converted (forgotten) to number,
// it is reasonable that toNumber is here.
export function toNumber(yen: Yen): number {
  return coreSchema.parse(yen)
}
```

OK: WorldGroup.ts
-- `World` is not the subject, but it is a concept that depends on `WorldGroup`, so it is OK to put it there.
-- However, if the number of descriptions for `World` increases in this file, please consider splitting the file into World.ts.

```typescript
import { z } from 'zod'

export const worldSchema = z.object({
  uid: z.string(),
  name: z.string(),
  author: z.string(),
  description: z.string(),
  imageUrl: z.string(),
  capacity: z.number(),
  tags: z.array(z.string()),
  publishedAt: z.string(),
  updatedAt: z.string(),
  dataSize: z.number(),
  supportedOS: z.union([
    z.literal('Windows'),
    z.literal('Android'),
    z.literal('iOS'),
  ]),
})
export type World = z.infer<typeof worldSchema>

export const worldGroupSchema = z.object({
  uid: z.string(),
  name: z.string(),
  worlds: z.array(worldSchema),
})
export type WorldGroup = z.infer<typeof worldGroupSchema>
```

NG: FooAndBarItem.ts
-- This should be split into Foo.ts and Bar.ts,
-- and `createBazFromFooAndBar()` should be placed in Baz.ts,
-- or in utils/Baz.ts.

```typescript
import { z } from 'zod'
import type { Baz } from '@/models/Baz'

export const fooSchema = /* some structure */
export type Foo = z.infer<typeof fooSchema>

export const barSchema = /* some structure */
export type Bar = z.infer<typeof barSchema>

export function createBazFromFooAndBar(foo: Foo, bar: Bar): Baz {
  return /* some process that uses foo and bar to create baz */
}
```

#### app/ and components/

- All `components/atoms/**/*.tsx` and `components/molecules/**/*.tsx` should not have state
  - Pass as props if some parameters are needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aiya000/claude-code-mobile-ssh](https://github.com/aiya000/claude-code-mobile-ssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
