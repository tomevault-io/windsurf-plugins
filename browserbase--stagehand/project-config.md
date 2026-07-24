---
trigger: always_on
description: This is a project that uses Stagehand V3, a browser automation framework with AI-powered `act`, `extract`, `observe`, and `agent` methods.
---

# Stagehand Project

This is a project that uses Stagehand V3, a browser automation framework with AI-powered `act`, `extract`, `observe`, and `agent` methods.

The main class can be imported as `Stagehand` from `@browserbasehq/stagehand`.

**Key Classes:**

- `Stagehand`: Main orchestrator class providing `act`, `extract`, `observe`, and `agent` methods
- `context`: A `V3Context` object that manages browser contexts and pages
- `page`: Individual page objects accessed via `stagehand.context.pages()[i]` or created with `stagehand.context.newPage()`

## Initialize

```typescript
import { Stagehand } from "@browserbasehq/stagehand";

const stagehand = new Stagehand({
  env: "LOCAL", // or "BROWSERBASE"
  verbose: 2, // 0, 1, or 2
  model: "openai/gpt-4.1-mini", // or any supported model
});

await stagehand.init();

// Access the browser context and pages
const page = stagehand.context.pages()[0];
const context = stagehand.context;

// Create new pages if needed
const page2 = await stagehand.context.newPage();
```

## Act

Actions are called on the `stagehand` instance (not the page). Use atomic, specific instructions:

```typescript
// Act on the current active page
await stagehand.act("click the sign in button");

// Act on a specific page (when you need to target a page that isn't currently active)
await stagehand.act("click the sign in button", { page: page2 });
```

**Important:** Act instructions should be atomic and specific:

- ✅ Good: "Click the sign in button" or "Type 'hello' into the search input"
- ❌ Bad: "Order me pizza" or "Type in the search bar and hit enter" (multi-step)

### Observe + Act Pattern (Recommended)

Cache the results of `observe` to avoid unexpected DOM changes:

```typescript
const instruction = "Click the sign in button";

// Get candidate actions
const actions = await stagehand.observe(instruction);

// Execute the first action
await stagehand.act(actions[0]);
```

To target a specific page:

```typescript
const actions = await stagehand.observe("select blue as the favorite color", {
  page: page2,
});
await stagehand.act(actions[0], { page: page2 });
```

## Extract

Extract data from pages using natural language instructions. The `extract` method is called on the `stagehand` instance.

### Basic Extraction (with schema)

```typescript
import { z } from "zod";

// Extract with explicit schema
const data = await stagehand.extract(
  "extract all apartment listings with prices and addresses",
  z.object({
    listings: z.array(
      z.object({
        price: z.string(),
        address: z.string(),
      }),
    ),
  }),
);

console.log(data.listings);
```

### Simple Extraction (without schema)

```typescript
// Extract returns a default object with 'extraction' field
const result = await stagehand.extract("extract the sign in button text");

console.log(result);
// Output: { extraction: "Sign in" }

// Or destructure directly
const { extraction } = await stagehand.extract(
  "extract the sign in button text",
);
console.log(extraction); // "Sign in"
```

### Targeted Extraction

Extract data from a specific element using a selector:

```typescript
const reason = await stagehand.extract(
  "extract the reason why script injection fails",
  z.string(),
  { selector: "/html/body/div[2]/div[3]/iframe/html/body/p[2]" },
);
```

### URL Extraction

When extracting links or URLs, use `z.string().url()`:

```typescript
const { links } = await stagehand.extract(
  "extract all navigation links",
  z.object({
    links: z.array(z.string().url()),
  }),
);
```

### Extracting from a Specific Page

```typescript
// Extract from a specific page (when you need to target a page that isn't currently active)
const data = await stagehand.extract(
  "extract the placeholder text on the name field",
  { page: page2 },
);
```

## Observe

Plan actions before executing them. Returns an array of candidate actions:

```typescript
// Get candidate actions on the current active page
const [action] = await stagehand.observe("Click the sign in button");

// Execute the action
await stagehand.act(action);
```

Observing on a specific page:

```typescript
// Target a specific page (when you need to target a page that isn't currently active)
const actions = await stagehand.observe("find the next page button", {
  page: page2,
});
await stagehand.act(actions[0], { page: page2 });
```

## Agent

Use the `agent` method to autonomously execute complex, multi-step tasks.

### Basic Agent Usage

```typescript
const page = stagehand.context.pages()[0];
await page.goto("https://www.google.com");

const agent = stagehand.agent({
  model: "google/gemini-2.0-flash",
  executionModel: "google/gemini-2.0-flash",
});

const result = await agent.execute({
  instruction: "Search for the stock price of NVDA",
  maxSteps: 20,
});

console.log(result.message);
```

### Computer Use Agent (CUA)

For more advanced scenarios using computer-use models:

```typescript
const agent = stagehand.agent({
  mode: "cua", // Enable Computer Use Agent mode
  model: "anthropic/claude-sonnet-4-20250514",
  // or "google/gemini-2.5-computer-use-preview-10-2025"
  systemPrompt: `You are a helpful assistant that can use a web browser.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [browserbase/stagehand](https://github.com/browserbase/stagehand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
