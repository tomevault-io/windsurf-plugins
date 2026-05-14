---
trigger: always_on
description: This app uses Clerk Billing to manage B2C SaaS subscriptions and feature access. All billing, payment processing, and subscription management is handled through Clerk's integrated billing system with Stripe.
---

# Clerk Billing & Subscription Management

## Overview
This app uses Clerk Billing to manage B2C SaaS subscriptions and feature access. All billing, payment processing, and subscription management is handled through Clerk's integrated billing system with Stripe.

## Available Plans & Features

### Subscription Plans
- **`free_user`**: Default free tier with limited features
- **`pro`**: Premium subscription with full feature access

### Available Features
- **`3_deck_limit`**: Free users can create up to 3 flashcard decks
- **`unlimited_decks`**: Pro users can create unlimited flashcard decks
- **`ai_flashcard_generation`**: Pro users can generate flashcards using AI

## Access Control Implementation

### Server-Side Protection with `has()` Method
**MANDATORY**: Use the `has()` method from Clerk's `auth()` for server-side access control in Server Components and Server Actions.

#### Plan-Based Protection
```typescript
import { auth } from '@clerk/nextjs/server';

export default async function DashboardPage() {
  const { has } = await auth();
  
  // Check if user has pro plan
  const hasProPlan = has({ plan: 'pro' });
  const isFreeUser = has({ plan: 'free_user' });
  
  return (
    <div>
      {hasProPlan && <ProFeatures />}
      {isFreeUser && <UpgradePrompt />}
    </div>
  );
}
```

#### Feature-Based Protection
```typescript
import { auth } from '@clerk/nextjs/server';

export default async function CreateDeckPage() {
  const { has } = await auth();
  
  // Check for specific features
  const hasUnlimitedDecks = has({ feature: 'unlimited_decks' });
  const hasAIGeneration = has({ feature: 'ai_flashcard_generation' });
  const hasThreeDeckLimit = has({ feature: '3_deck_limit' });
  
  return (
    <div>
      {hasAIGeneration && <AIGenerationButton />}
      {hasThreeDeckLimit && <DeckLimitWarning />}
    </div>
  );
}
```

### Client-Side Protection with `<Protect>` Component
**MANDATORY**: Use the `<Protect>` component for conditional rendering in Client Components.

#### Plan-Based Component Protection
```typescript
import { Protect } from '@clerk/nextjs';

export function ProFeatureSection() {
  return (
    <Protect
      plan="pro"
      fallback={
        <div className="p-4 border-2 border-dashed border-gray-300 rounded-lg">
          <p className="text-center text-muted-foreground">
            Upgrade to Pro to unlock this feature
          </p>
        </div>
      }
    >
      <AIFlashcardGenerator />
    </Protect>
  );
}
```

#### Feature-Based Component Protection
```typescript
import { Protect } from '@clerk/nextjs';

export function CreateDeckButton() {
  return (
    <Protect
      feature="unlimited_decks"
      fallback={
        <Button disabled variant="outline">
          Upgrade to create more decks
        </Button>
      }
    >
      <Button>Create New Deck</Button>
    </Protect>
  );
}
```

## Flashcard App-Specific Patterns

### Deck Creation Limits
**MANDATORY**: Enforce deck creation limits based on user plan and existing deck count.

```typescript
// In src/db/queries/decks.ts
import { auth } from '@clerk/nextjs/server';

export async function canCreateDeck(): Promise<{ canCreate: boolean; reason?: string }> {
  const { has, userId } = await auth();
  if (!userId) return { canCreate: false, reason: "Unauthorized" };
  
  const hasUnlimitedDecks = has({ feature: 'unlimited_decks' });
  
  if (hasUnlimitedDecks) {
    return { canCreate: true };
  }
  
  // Check deck count for free users
  const deckCount = await db.select({ count: sql<number>`count(*)` })
    .from(decksTable)
    .where(eq(decksTable.userId, userId));
    
  const currentCount = deckCount[0]?.count || 0;
  
  if (currentCount >= 3) {
    return { 
      canCreate: false, 
      reason: "Free users can only create 3 decks. Upgrade to Pro for unlimited decks." 
    };
  }
  
  return { canCreate: true };
}
```

### AI Feature Protection
**MANDATORY**: Protect AI flashcard generation features for Pro users only.

```typescript
// In server actions
"use server";

import { auth } from '@clerk/nextjs/server';

export async function generateAIFlashcards(prompt: string) {
  const { has, userId } = await auth();
  if (!userId) throw new Error("Unauthorized");
  
  const hasAIFeature = has({ feature: 'ai_flashcard_generation' });
  
  if (!hasAIFeature) {
    throw new Error("AI flashcard generation requires a Pro subscription");
  }
  
  // Proceed with AI generation
  return await generateFlashcardsWithAI(prompt);
}
```

### Dashboard Feature Differentiation
**MANDATORY**: Show different UI elements based on user subscription level.

```typescript
import { auth } from '@clerk/nextjs/server';
import { Protect } from '@clerk/nextjs';

export default async function DashboardPage() {
  const { has } = await auth();
  const userDecks = await getUserDecks();
  
  const hasUnlimitedDecks = has({ feature: 'unlimited_decks' });
  const deckCount = userDecks.length;
  
  return (
    <div>
      <div className="flex justify-between items-center">
        <h1>My Flashcard Decks ({deckCount})</h1>
        
        <Protect
          feature="unlimited_decks"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomphill/flashycardycourse](https://github.com/tomphill/flashycardycourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
