---
trigger: always_on
description: You are helping develop a **Motia project** - a unified backend framework that uses event-driven architecture with multiple programming languages.
---

# Motia Framework Development Assistant

You are helping develop a **Motia project** - a unified backend framework that uses event-driven architecture with multiple programming languages.

## Core Motia Concepts

### Steps Architecture

- **Steps** are the fundamental building blocks - each has `config` and `handler`
- **API Steps**: HTTP endpoints (`type: 'api'`)
- **Event Steps**: Event processors (`type: 'event'`)
- **Cron Steps**: Scheduled tasks (`type: 'cron'`)
- **Stream Steps**: Real-time data (`type: 'stream'`)
- **NOOP Steps**: Workflow routing (`type: 'noop'`)

### Event-Driven Communication

- Steps communicate via `emit({ topic: 'event-name', data: {...} })`
- Subscribe to events: `subscribes: ['topic-name']` in config
- Creates loose coupling and parallel execution

### State Management

- Persistent state via `state.set(group, key, value)` and `state.get(group, key)`
- State is scoped by groups ('orders', 'users', etc.) and keys
- Supports trace-scoped and global persistence

## File Structure

```
steps/              # All step implementations
├── *.step.ts      # Step files (config + handler)
├── *-features.json # Tutorial/workbench metadata
services/          # Shared business logic
types.d.ts         # Auto-generated types from step configs
motia-workbench.json # Visual flow configuration
```

## Step Implementation Patterns

Motia supports **multiple programming languages** in the same project. Choose the best language for each task:

- **TypeScript/JavaScript**: APIs, web logic, real-time features
- **Python**: AI/ML, data science, image processing, analytics
- **Ruby**: Reports, data exports, file processing, templating

### TypeScript API Step

```typescript
import { z } from 'zod'
import type { ApiRouteConfig, Handlers } from '@motia/core'

export const config: ApiRouteConfig = {
  type: 'api',
  name: 'CreateOrder',
  method: 'POST',
  path: '/orders',
  bodySchema: z.object({
    productId: z.string(),
    quantity: z.number(),
  }),
  emits: ['order.created'],
  flows: ['ecommerce'],
}

export const handler: Handlers['CreateOrder'] = async (req, { logger, emit, state, traceId }) => {
  const order = { id: crypto.randomUUID(), ...req.body, createdAt: new Date() }
  await state.set('orders', order.id, order)
  await emit({ topic: 'order.created', data: order })
  return { status: 201, body: order }
}
```

### JavaScript Event Step

```javascript
// steps/process-payment.step.js
exports.config = {
  type: 'event',
  name: 'ProcessPayment',
  subscribes: ['order.created'],
  emits: ['payment.processed', 'payment.failed'],
  input: {
    id: 'string',
    amount: 'number',
    currency: 'string',
  },
}

exports.handler = async (order, { logger, emit, state }) => {
  try {
    logger.info('Processing payment', { orderId: order.id })

    // Simulate payment processing
    const paymentResult = await processPayment(order)

    await state.set('payments', order.id, paymentResult)
    await emit({
      topic: 'payment.processed',
      data: { orderId: order.id, paymentId: paymentResult.id },
    })
  } catch (error) {
    logger.error('Payment failed', { orderId: order.id, error: error.message })
    await emit({ topic: 'payment.failed', data: { orderId: order.id, error: error.message } })
  }
}
```

### Python AI/ML Event Step

```python
# steps/analyze_sentiment.step.py
import asyncio
from transformers import pipeline

config = {
    "type": "event",
    "name": "AnalyzeSentiment",
    "subscribes": ["review.submitted"],
    "emits": ["sentiment.analyzed"],
    "input": {
        "reviewId": "string",
        "text": "string",
        "userId": "string"
    }
}

# Initialize ML model
sentiment_analyzer = pipeline("sentiment-analysis")

async def handler(review_data, context):
    logger = context["logger"]
    emit = context["emit"]
    state = context["state"]

    try:
        logger.info(f"Analyzing sentiment for review {review_data['reviewId']}")

        # Run sentiment analysis
        result = sentiment_analyzer(review_data["text"])
        sentiment_score = result[0]["score"]
        sentiment_label = result[0]["label"]

        analysis = {
            "reviewId": review_data["reviewId"],
            "sentiment": sentiment_label,
            "confidence": sentiment_score,
            "analyzedAt": datetime.now().isoformat()
        }

        await state.set("sentiment_analyses", review_data["reviewId"], analysis)
        await emit({
            "topic": "sentiment.analyzed",
            "data": analysis
        })

    except Exception as error:
        logger.error(f"Sentiment analysis failed: {str(error)}")
        await emit({
            "topic": "analysis.failed",
            "data": {"reviewId": review_data["reviewId"], "error": str(error)}
        })
```

### Python Data Processing Step

```python
# steps/generate_analytics.step.py
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

config = {
    "type": "cron",
    "name": "GenerateAnalytics",
    "cron": "0 0 * * 1",  # Weekly on Monday
    "emits": ["analytics.generated"]
}

async def handler(context):
    logger = context["logger"]
    emit = context["emit"]
    state = context["state"]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MotiaDev/motia-examples](https://github.com/MotiaDev/motia-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
