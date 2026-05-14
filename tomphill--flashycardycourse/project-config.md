---
trigger: always_on
description: This project follows strict patterns for data handling to ensure security, type safety, and performance. All data operations must follow these established patterns.
---

# Data Handling Patterns

## Overview
This project follows strict patterns for data handling to ensure security, type safety, and performance. All data operations must follow these established patterns.

## 🚨 MANDATORY: Centralized Query Functions

### Database Query Organization
**CRITICAL REQUIREMENT**: ALL database operations (SELECT, INSERT, UPDATE, DELETE) MUST be performed through dedicated helper functions located in the `src/db/queries/` directory. NEVER perform direct database operations in components, server actions, or API routes.

#### Directory Structure
```
src/db/queries/
├── decks.ts      # Deck-related queries
├── cards.ts      # Card-related queries
└── index.ts      # Re-export all queries
```

#### Query Function Pattern
Every query function MUST follow this exact pattern:

```typescript
// src/db/queries/decks.ts
import { auth } from "@clerk/nextjs/server";
import { db } from '@/db';
import { decksTable } from '@/db/schema';
import { eq, and } from 'drizzle-orm';

export async function getUserDecks() {
  const { userId } = await auth();
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  return await db.select()
    .from(decksTable)
    .where(eq(decksTable.userId, userId));
}

export async function getDeckById(deckId: string) {
  const { userId } = await auth();
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  const deck = await db.select()
    .from(decksTable)
    .where(
      and(
        eq(decksTable.id, deckId),
        eq(decksTable.userId, userId)
      )
    )
    .limit(1);
    
  return deck[0] || null;
}

export async function createDeck(data: { title: string; description?: string }) {
  const { userId } = await auth();
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  const newDeck = await db.insert(decksTable)
    .values({
      ...data,
      userId,
    })
    .returning();
    
  return newDeck[0];
}

export async function updateDeck(deckId: string, data: { title?: string; description?: string }) {
  const { userId } = await auth();
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  const updatedDeck = await db.update(decksTable)
    .set(data)
    .where(
      and(
        eq(decksTable.id, deckId),
        eq(decksTable.userId, userId)
      )
    )
    .returning();
    
  return updatedDeck[0] || null;
}

export async function deleteDeck(deckId: string) {
  const { userId } = await auth();
  if (!userId) {
    throw new Error("Unauthorized");
  }
  
  await db.delete(decksTable)
    .where(
      and(
        eq(decksTable.id, deckId),
        eq(decksTable.userId, userId)
      )
    );
}
```

#### Query Re-exports
**MANDATORY**: Create an index file to re-export all queries:

```typescript
// src/db/queries/index.ts
export * from './decks';
export * from './cards';
```

## Data Retrieval Rules

### Server Components with Query Functions
**MANDATORY**: All data retrieval operations MUST be performed in Server Components using the centralized query functions.

```typescript
// ✅ CORRECT: Data fetching in Server Component using query functions
import { getUserDecks } from '@/db/queries';

export default async function DashboardPage() {
  // Use centralized query function
  const userDecks = await getUserDecks();
    
  return <DecksList decks={userDecks} />;
}
```

```typescript
// ❌ WRONG: Direct database operations in components
import { auth } from "@clerk/nextjs/server";
import { db } from '@/db';
import { decksTable } from '@/db/schema';
import { eq } from 'drizzle-orm';

export default async function DashboardPage() {
  // ❌ This violates the centralized query rule
  const { userId } = await auth();
  if (!userId) redirect('/');
  
  const userDecks = await db.select()
    .from(decksTable)
    .where(eq(decksTable.userId, userId));
    
  return <DecksList decks={userDecks} />;
}
```

```typescript
// ❌ WRONG: Never fetch data in Client Components
"use client";
import { useEffect, useState } from 'react';

export default function DashboardPage() {
  const [decks, setDecks] = useState([]);
  
  useEffect(() => {
    // ❌ This violates the server component rule
    fetch('/api/decks').then(res => res.json()).then(setDecks);
  }, []);
  
  return <DecksList decks={decks} />;
}
```

## Database Mutations Rules

### Server Actions with Query Functions
**MANDATORY**: All database mutations MUST be performed via Server Actions that use the centralized query functions.

```typescript
// ✅ CORRECT: Server Actions using centralized query functions
"use server";

import { createDeck, updateDeck, deleteDeck } from '@/db/queries';
import { z } from 'zod';

const CreateDeckSchema = z.object({
  title: z.string().min(1, "Title is required"),
  description: z.string().optional(),
});

type CreateDeckInput = z.infer<typeof CreateDeckSchema>;

export async function createDeckAction(input: CreateDeckInput) {
  // Validate input with Zod
  const validatedInput = CreateDeckSchema.parse(input);
  
  // Use centralized query function
  return await createDeck(validatedInput);
}

export async function updateDeckAction(deckId: string, input: Partial<CreateDeckInput>) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomphill/flashycardycourse](https://github.com/tomphill/flashycardycourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
