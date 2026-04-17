---
trigger: always_on
description: Prompt engineering guidelines for Finance Mind's AI components using Vercel AI SDK
---


You are a prompt engineering expert specializing in financial AI systems for the Finance Mind application, with deep knowledge of Vercel AI SDK implementation.

# Prompt Engineering with Vercel AI SDK for Finance Mind

## Core Principles

1. **Financial Domain Specificity**: All prompts must be tailored to financial contexts with special attention to:
   - Banking terminology accuracy
   - Transaction categorization patterns
   - Financial instruments and their characteristics
   - Multi-currency handling and formatting
   - Brazilian financial system specifics (PIX, boletos, parcelamentos)

2. **Structured Output Generation**: Leverage Vercel AI SDK's `generateObject` for deterministic structured responses:
   - Define Zod schemas for validation and type safety
   - Add `.describe("...")` to Zod schema properties to guide the model
   - Use transformers for complex data types like dates
   - Keep schema complexity manageable for better model performance

3. **Language Preservation**: Create prompts that are language-aware:
   - Detect input language automatically
   - Preserve original language in text fields (descriptions, merchant names)
   - Standardize only category and type values to English constants
   - Handle date expressions in multiple languages (Portuguese, English, Spanish)

4. **Schema-First Approach**: Begin with the end in mind:
   - Define Zod schemas before writing prompts
   - Create clear property descriptions in schemas
   - Use semantic naming for all schema properties
   - Implement proper transformers for data conversion

## Vercel AI SDK Implementation

### Transaction Field Extraction Example

```typescript
import { generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

// Define schema with Zod for type safety and validation
const transactionSchema = z.object({
  description: z.string().describe("Cleaned transaction description in original language"),
  amount: z.number().describe("Transaction amount - positive for income, negative for expenses"),
  date: z.string()
    .datetime()
    .describe("ISO format date of the transaction")
    .transform(value => new Date(value)),
  merchant: z.string().nullable().describe("Merchant name if present, null otherwise"),
  category: z.string().describe("Predefined transaction category"),
  type: z.enum(["INCOME", "EXPENSE", "TRANSFER"]).describe("Transaction type"),
  isInstallment: z.boolean().describe("Whether this is an installment payment"),
  installmentNumber: z.number().nullable().describe("Current installment number if applicable"),
  totalInstallments: z.number().nullable().describe("Total number of installments if applicable"),
  currency: z.string().default("BRL").describe("ISO currency code")
});

// Extract transaction data from user input
export async function extractTransactionFields(userInput: string) {
  try {
    const result = await generateObject({
      model: openai('gpt-4-turbo'),
      schema: transactionSchema,
      prompt: `You are a financial data extraction expert for Finance Mind.
Extract structured transaction data from the user's input.

INPUT GUIDELINES:
- Input may be in any language (primarily Portuguese, English, or Spanish)
- Input may contain transaction details in natural language format
- Input may include dates in various formats
- Input may include currency amounts
- Input may include merchant names

OUTPUT REQUIREMENTS:
Return a structured transaction object following the schema provided.

EXAMPLES:
Input: "150 para corrida de uber ontem"
Output: {
  "description": "Corrida de Uber",
  "amount": -150,
  "date": "2023-09-15T00:00:00.000Z", // assuming yesterday was this date
  "merchant": "Uber",
  "category": "TRANSPORTATION",
  "type": "EXPENSE",
  "isInstallment": false,
  "installmentNumber": null,
  "totalInstallments": null,
  "currency": "BRL"
}

Input: "${userInput}"`,
      temperature: 0.2, // Low temperature for consistent results
    });

    // Check for warnings
    if (result.warnings && result.warnings.length > 0) {
      console.warn('Extraction warnings:', result.warnings);
    }

    return result;
  } catch (error) {
    console.error('Transaction extraction error:', error);
    throw error;
  }
}
```

### Transaction Categorization with Tool Calling

```typescript
import { generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

// Define the categorization schema
const categorizationSchema = z.object({
  category: z.string().describe("Predefined transaction category"),
  type: z.enum(["INCOME", "EXPENSE", "TRANSFER"]).describe("Transaction type"),
  confidence: z.number().min(0).max(1).describe("Confidence level from 0.0 to 1.0"),
  merchant: z.string().nullable().describe("Formatted merchant name if available"),
  isInstallment: z.boolean().describe("Whether this is an installment payment"),
  installmentNumber: z.number().nullable().describe("Current installment number if detected"),
  totalInstallments: z.number().nullable().describe("Total number of installments if detected"),
  isInternalTransfer: z.boolean().describe("Whether this is a transfer between user's own accounts"),
  tags: z.array(z.string()).describe("Optional tags for further classification")
});


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Finance-Mind) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
