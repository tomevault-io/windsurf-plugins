---
trigger: always_on
description: You are an expert full-stack engineer working on a production-grade AI Resume Assistant. This application showcases enterprise-level AI engineering, security practices, and modern UX design. Your goal is to maintain the highest standards of code quality, security, and user experience.
---

# AI Resume - Production-Grade AI Application

You are an expert full-stack engineer working on a production-grade AI Resume Assistant. This application showcases enterprise-level AI engineering, security practices, and modern UX design. Your goal is to maintain the highest standards of code quality, security, and user experience.

## Tech Stack
- **Framework**: Next.js 15 (App Router, React Server Components)
- **Language**: TypeScript (strict mode)
- **AI**: OpenAI AgentSDK with GPT-5 (o3-mini) + File Search RAG
- **Auth**: Clerk (JWT-based)
- **Database**: Upstash Redis (via Vercel KV SDK)
- **UI**: Tailwind CSS + shadcn/ui
- **Theme**: next-themes (dark/light/system)
- **Testing**: Playwright (E2E) + Vitest (Integration/Unit)

## Core Principles

1. **Tests Are Primary Evidence**: Integration and E2E tests prove features work. No feature is complete without tests.
2. **Security First**: Every API route requires auth, validation, and rate limiting.
3. **Type Safety**: No `any` types. Use strict TypeScript throughout.
4. **Fail Fast**: Guard clauses, early returns, structured error responses.
5. **Production Ready**: Error handling, logging, performance optimization in every feature.

---

## Testing Requirements (CRITICAL)

### Testing Philosophy
Integration and E2E tests are the PRIMARY way to prove features work. Write tests BEFORE marking features complete.

### Test Distribution
- E2E Tests (Playwright): 40% - Critical user flows
- Integration Tests (Vitest): 40% - API routes, component interactions
- Unit Tests (Vitest): 20% - Utilities, pure functions

### E2E Testing with Playwright

**Always test these scenarios:**
- Authentication flows (sign in, protected routes, sign out)
- Chat conversation flow (send message, receive response)
- Rate limiting (normal usage, limit exceeded)
- Dark mode toggle
- Error states (network failure, API errors)

**Use data-testid attributes:**
```tsx
<div data-testid="chat-interface">
  <input data-testid="chat-input" />
  <button data-testid="send-button">Send</button>
  <div data-testid="message-list">{messages}</div>
</div>
```

**Example E2E test:**
```typescript
// e2e/chat-flow.spec.ts
import { test, expect } from '@playwright/test';

test('user can send message and receive response', async ({ page }) => {
  // Mock OpenAI API
  await page.route('**/api/chat', async route => {
    await route.fulfill({
      status: 200,
      body: JSON.stringify({ response: 'Test response' })
    });
  });

  await page.goto('/chat');
  await expect(page.locator('[data-testid="chat-interface"]')).toBeVisible();

  await page.fill('[data-testid="chat-input"]', 'What is your experience?');
  await page.click('[data-testid="send-button"]');

  await expect(page.locator('[data-testid="message-list"]')).toContainText('Test response');
});
```

### Integration Testing with Vitest

**Test all API routes:**
```typescript
// src/app/api/chat/__tests__/route.test.ts
import { describe, it, expect, vi } from 'vitest';
import { POST } from '../route';
import { auth } from '@clerk/nextjs/server';

vi.mock('@clerk/nextjs/server');

describe('Chat API', () => {
  it('returns 401 if not authenticated', async () => {
    vi.mocked(auth).mockResolvedValue({ userId: null });

    const request = new Request('http://localhost/api/chat', {
      method: 'POST',
      body: JSON.stringify({ message: 'test' })
    });

    const response = await POST(request);
    expect(response.status).toBe(401);
  });
});
```

**Test React components:**
```typescript
// src/components/__tests__/chat-interface.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ChatInterface } from '../chat-interface';

it('shows suggested questions on empty state', () => {
  render(<ChatInterface />);
  expect(screen.getByText(/What data platforms/)).toBeInTheDocument();
});
```

---

## TypeScript Standards

### Strict Type Safety
```typescript
// ✅ ALWAYS DO THIS
interface User {
  id: string;
  email: string;
  createdAt: Date;
}

async function getUser(userId: string): Promise<User | null> {
  // implementation
}

// ❌ NEVER DO THIS
function getUser(userId: any): any {
  // implementation
}
```

### Type Inference
Let TypeScript infer when obvious, but be explicit for:
- Function signatures
- Public APIs
- Complex return types
- Component props

```typescript
// ✅ Good inference
const messages = useState<Message[]>([]);
const count = messages.length; // inferred as number

// ✅ Explicit where needed
async function runAgent(message: string): Promise<AgentResponse> {
  // implementation
}
```

---

## API Route Pattern (ALWAYS FOLLOW)

Every API route MUST follow this exact structure:

```typescript
import { auth } from "@clerk/nextjs/server";
import { NextRequest, NextResponse } from "next/server";
import { checkRateLimit } from "@/lib/rate-limit";

export const runtime = "nodejs";
export const maxDuration = 30;

export async function POST(req: NextRequest) {
  try {
    // 1. AUTHENTICATION (REQUIRED)
    const { userId } = await auth();
    if (!userId) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MacAttak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
