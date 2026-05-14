---
trigger: always_on
description: This project uses the **Vercel AI SDK** (`ai` npm package) to power AI-driven flashcard generation features. The AI integration allows users to automatically generate flashcards from text input, topics, or learning materials.
---

# Vercel AI Integration for Flashcard Generation

## Overview
This project uses the **Vercel AI SDK** (`ai` npm package) to power AI-driven flashcard generation features. The AI integration allows users to automatically generate flashcards from text input, topics, or learning materials.

## Core Dependencies
```json
{
  "ai": "^3.x.x",
  "@ai-sdk/openai": "^0.x.x",
  "zod": "^3.x.x"
}
```

## AI Feature Architecture

### Authentication & Billing Integration
**CRITICAL**: AI flashcard generation is a **PREMIUM FEATURE** that requires:
- User authentication via Clerk
- Pro subscription (`plan: 'pro'` or `feature: 'ai_flashcard_generation'`)
- Proper feature gating using Clerk's billing system

```typescript
import { auth } from '@clerk/nextjs/server';

export async function generateAIFlashcards(prompt: string) {
  const { has, userId } = await auth();
  if (!userId) throw new Error("Unauthorized");
  
  const hasAIFeature = has({ feature: 'ai_flashcard_generation' });
  if (!hasAIFeature) {
    throw new Error("AI flashcard generation requires a Pro subscription");
  }
  
  // Proceed with AI generation...
}
```

### Structured Data Generation Pattern
**MANDATORY**: Use Vercel AI's `generateObject` function for creating structured flashcard data.

#### Core Implementation Pattern
```typescript
import { generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

// Define Zod schema for flashcard structure
const FlashcardSchema = z.object({
  flashcards: z.array(z.object({
    front: z.string().min(1, "Front content required"),
    back: z.string().min(1, "Back content required"),
    difficulty: z.enum(['easy', 'medium', 'hard']).optional(),
    tags: z.array(z.string()).optional(),
  })),
  metadata: z.object({
    topic: z.string(),
    totalCards: z.number(),
    estimatedStudyTime: z.number().optional(),
  }).optional(),
});

type FlashcardGenerationResult = z.infer<typeof FlashcardSchema>;

export async function generateFlashcardsWithAI(
  prompt: string, 
  cardCount: number = 10,
  difficulty: 'easy' | 'medium' | 'hard' = 'medium'
): Promise<FlashcardGenerationResult> {
  const { object } = await generateObject({
    model: openai('gpt-4o-mini'), // Cost-effective model for flashcards
    schema: FlashcardSchema,
    prompt: `Generate ${cardCount} flashcards about: ${prompt}
    
    Requirements:
    - Create exactly ${cardCount} flashcards
    - Difficulty level: ${difficulty}
    - Front: Clear, concise questions or prompts
    - Back: Accurate, detailed answers or explanations
    - Cover different aspects of the topic
    - Ensure educational value and accuracy
    - Make questions progressively challenging if requested
    
    Topic: ${prompt}`,
    temperature: 0.7, // Balanced creativity and consistency
  });
  
  return object;
}
```

### Server Action Integration
**MANDATORY**: AI generation MUST be implemented as Server Actions following the project's data handling patterns.

```typescript
// src/app/decks/[deckId]/actions.ts (or similar)
"use server";

import { auth } from '@clerk/nextjs/server';
import { createCard } from '@/db/queries';
import { generateFlashcardsWithAI } from '@/lib/ai';
import { z } from 'zod';

const AIGenerationSchema = z.object({
  deckId: z.string().uuid(),
  prompt: z.string().min(10, "Prompt must be at least 10 characters"),
  cardCount: z.number().min(1).max(50).default(10),
  difficulty: z.enum(['easy', 'medium', 'hard']).default('medium'),
});

type AIGenerationInput = z.infer<typeof AIGenerationSchema>;

export async function generateAIFlashcardsAction(input: AIGenerationInput) {
  // 1. Validate input
  const validatedInput = AIGenerationSchema.parse(input);
  
  // 2. Check authentication and billing
  const { has, userId } = await auth();
  if (!userId) throw new Error("Unauthorized");
  
  const hasAIFeature = has({ feature: 'ai_flashcard_generation' });
  if (!hasAIFeature) {
    throw new Error("AI flashcard generation requires a Pro subscription");
  }
  
  // 3. Verify deck ownership (using centralized query)
  const deck = await getDeckById(validatedInput.deckId);
  if (!deck) {
    throw new Error("Deck not found or unauthorized");
  }
  
  // 4. Generate flashcards with AI
  const result = await generateFlashcardsWithAI(
    validatedInput.prompt,
    validatedInput.cardCount,
    validatedInput.difficulty
  );
  
  // 5. Save generated cards to database (using centralized queries)
  const createdCards = [];
  for (const flashcard of result.flashcards) {
    const card = await createCard({
      deckId: validatedInput.deckId,
      front: flashcard.front,
      back: flashcard.back,
    });
    createdCards.push(card);
  }
  
  return {
    cards: createdCards,
    metadata: result.metadata,
  };
}
```

## UI Integration Patterns

### Premium Feature Protection
**MANDATORY**: Always wrap AI features with proper billing protection using Clerk's `<Protect>` component.

```typescript
import { Protect } from '@clerk/nextjs';
import { Button } from '@/components/ui/button';

export function AIGenerationButton({ deckId }: { deckId: string }) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomphill/flashycardycourse](https://github.com/tomphill/flashycardycourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
