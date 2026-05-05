---
trigger: always_on
description: This document explains how to integrate Sainsbury's CLI into AI agent frameworks.
---

## Agent Integration Guide

This document explains how to integrate Sainsbury's CLI into AI agent frameworks.

---

## Supported Frameworks

- ✅ **OpenClaw** / **Clawdbot** - Skills system
- ✅ **Pi Agent** / **Mom** - Slack bot with skills
- ✅ **Claude Desktop** - MCP server (future)
- ✅ **Custom agents** - Any framework that can call bash

---

## Quick Integration

### 1. Add as Skill

Copy to your agent's skills directory:

```bash
cp -r sainsburys-cli /path/to/agent/skills/
```

### 2. Agent Calls Commands

```typescript
// From your agent code
await bash("cd skills/sainsburys-cli && npm run groc search 'milk'");
```

### 3. Parse JSON Responses

```typescript
const stdout = await bash("cd skills/sainsburys-cli && npm run groc search 'milk' --json");
const results = JSON.parse(stdout);

results.products.forEach(product => {
  console.log(`${product.name} - £${product.retail_price.price}`);
});
```

---

## Skill File Format

The `SKILL.md` follows the open skills format used by OpenClaw, Pi, and other agent frameworks.

### Frontmatter

```yaml
---
name: sainsburys-groceries
description: AI-powered meal planning and grocery ordering
license: MIT
compatibility: Node.js 18+, TypeScript, Playwright
metadata:
  author: zish
  version: "2.0.0"
allowed-tools: Bash({baseDir}/node:*), Bash(npm:run:groc:*)
---
```

### Triggers

Agent should load this skill when user:
- Mentions meal planning or groceries
- Asks "what's for dinner?"
- Wants to order food/shopping
- Talks about recipes or cooking
- Mentions Sainsbury's

---

## Natural Language Workflow

### User Intent Detection

```typescript
const intents = {
  mealPlanning: ["plan meals", "what should I cook", "dinner ideas"],
  shopping: ["add to basket", "order groceries", "buy milk"],
  delivery: ["book slot", "delivery Tuesday", "checkout"],
  query: ["what's in my basket", "show orders", "search for bread"]
};

if (userMessage.match(/plan meals|what.*cook|dinner ideas/i)) {
  await startMealPlanning();
}
```

### Meal Planning Flow

```typescript
async function startMealPlanning() {
  // 1. Ask constraints
  await ask("How many people? Budget? Dietary restrictions?");
  
  // 2. Suggest meals
  const meals = await suggestMeals({
    people: 2,
    budget: 50,
    dietary: ["halal"]
  });
  
  // 3. Get approval
  await showMeals(meals);
  const approved = await waitForApproval();
  
  // 4. Generate shopping list
  const ingredients = extractIngredients(approved);
  
  // 5. Search products
  for (const ingredient of ingredients) {
    const result = await bash(`cd skills/sainsburys-cli && npm run groc search "${ingredient}" --json`);
    const products = JSON.parse(result);
    const best = pickBestMatch(products, ingredient);
    shoppingList.push(best);
  }
  
  // 6. Show list and add to basket
  await showShoppingList(shoppingList);
  if (await confirm("Add to basket?")) {
    for (const item of shoppingList) {
      await bash(`cd skills/sainsburys-cli && npm run groc add ${item.product_uid} --qty ${item.quantity}`);
    }
  }
  
  // 7. Checkout
  await bash(`cd skills/sainsburys-cli && npm run groc basket --json`);
  // ... show basket, book slot, checkout
}
```

---

## Block Kit Integration (Slack Bots)

For Slack agents like Pi/Mom, use Block Kit for rich UIs.

### Shopping List

```javascript
async function showShoppingList(items, total) {
  const blocks = [
    {
      "type": "header",
      "text": {"type": "plain_text", "text": "🛒 Your Shopping List"}
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": `*Total: £${total}* (${items.length} items)`
      }
    },
    {"type": "divider"},
    ...buildItemSections(items),
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {"type": "plain_text", "text": "Add All to Basket"},
          "action_id": "add_to_basket",
          "style": "primary"
        },
        {
          "type": "button",
          "text": {"type": "plain_text", "text": "Modify List"},
          "action_id": "modify_list"
        }
      ]
    }
  ];
  
  await sendBlocks(blocks);
}

function buildItemSections(items) {
  const categories = groupByCategory(items);
  const sections = [];
  
  for (const [category, products] of Object.entries(categories)) {
    const itemsText = products
      .map(p => `• ${p.name} - £${p.price}`)
      .join('\n');
    
    sections.push({
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": `*${category}*\n${itemsText}`
        }
      ]
    });
  }
  
  return sections;
}
```

### Basket Summary

```javascript
async function showBasket(basket) {
  const blocks = [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": `*🛒 Your Basket*\n\n*${basket.total_quantity} items* | *£${basket.total_cost}*`
      }
    },
    {"type": "divider"}
  ];
  
  // Group items
  basket.products.slice(0, 10).forEach(item => {
    blocks.push({
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": `*${item.quantity}x* ${item.name}\n£${item.unit_price} each`
      },
      "accessory": {
        "type": "button",
        "text": {"type": "plain_text", "text": "Remove"},

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abracadabra50/uk-grocery-cli](https://github.com/abracadabra50/uk-grocery-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
