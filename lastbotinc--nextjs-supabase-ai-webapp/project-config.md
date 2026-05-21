---
trigger: always_on
description: Description of unit testing design patterns and instructions
---

---
description: implementation pattern for unit tests with vitest
globs:
---

# Unit Testing Implementation Patterns (Vitest)

This document describes the patterns and best practices for implementing unit tests in our Next.js application using **Vitest**, covering both component testing and API route testing.

## Test Setup

### Directory Structure
(Remains the same)
```
app/
  components/
    Button/
      Button.tsx
      __tests__/
        Button.test.tsx
  api/
    languages/
      route.ts
      __tests__/
        route.test.ts
```

### Required Dependencies
Update your `package.json` devDependencies:
```json
{
  "devDependencies": {
    "@testing-library/jest-dom": "^6.4.0", // For DOM matchers
    "@testing-library/react": "^15.0.0",
    "@testing-library/user-event": "^14.5.0",
    "@vitejs/plugin-react": "^4.2.0", // For Vite + React
    "jsdom": "^24.0.0", // Test environment
    "vitest": "^1.5.0", // Test runner
    "vite-tsconfig-paths": "^4.3.0", // For resolving tsconfig paths
    "resize-observer-polyfill": "^1.5.1" // Polyfill for JSDOM
  }
}
```

### Vitest Configuration (`vitest.config.ts`)
```typescript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    environment: 'jsdom', // Use JSDOM for browser-like environment
    setupFiles: ['./vitest.setup.ts'], // Setup file for global configurations
    include: ['**/*.test.{ts,tsx}'], // Test file pattern
    globals: true, // Optional: Makes vi, expect, etc. globally available
  },
})
```

### Vitest Setup (`vitest.setup.ts`)
This file configures testing-library matchers and adds necessary polyfills.
```typescript
import '@testing-library/jest-dom'
import { expect, afterEach } from 'vitest'
import { cleanup } from '@testing-library/react'
import * as matchers from '@testing-library/jest-dom/matchers'
import 'resize-observer-polyfill' // Polyfill for ResizeObserver in JSDOM

// Extend Vitest's expect method with testing-library matchers
expect.extend(matchers)

// Cleanup after each test case
afterEach(() => {
  cleanup()
})
```

## Component Testing

### 1. Basic Component Test Structure
```typescript
// No environment pragma needed with vitest.config.ts
import { render, screen, fireEvent } from '@testing-library/react'
import '@testing-library/jest-dom' // Matchers are extended in setup
import { describe, it, expect, beforeEach, vi } from 'vitest' // Import from vitest
import { ComponentToTest } from '../ComponentToTest'

describe('ComponentToTest', () => {
  // Setup default props and mocks
  const defaultProps = {
    onAction: vi.fn() // Use vi.fn()
  }

  beforeEach(() => {
    vi.clearAllMocks() // Use vi.clearAllMocks()
  })

  it('renders correctly', () => {
    render(<ComponentToTest {...defaultProps} />)
    // Add assertions using expect() extended with jest-dom matchers
    expect(screen.getByText('Some Text')).toBeInTheDocument()
  })
})
```

### 2. Testing with Internationalization
(Pattern remains largely the same, ensure `vi.mock` is used if mocking `next-intl`)
```typescript
import { render, screen } from '@testing-library/react'
import { describe, it, expect, vi } from 'vitest'
import { NextIntlClientProvider } from 'next-intl'
import Component from '../Component' // Assume this is the component using translations

// Mock translations
const mockTranslations = {
  namespace: {
    key: 'Translated text'
  }
}

// Mock next-intl if needed (using vi.mock)
vi.mock('next-intl', async (importOriginal) => {
  const mod = await importOriginal<typeof import('next-intl')>()
  return {
    ...mod,
    useTranslations: () => (key: string) => mockTranslations.namespace[key] || key,
    // Mock other exports like NextIntlClientProvider if necessary,
    // but often letting the real provider work is fine for client components.
  }
})


// Helper function to render with translations
const renderWithTranslations = (component: React.ReactNode) => {
  return render(
    <NextIntlClientProvider messages={mockTranslations} locale="en">
      {component}
    </NextIntlClientProvider>
  )
}

it('renders translated content', () => {
  renderWithTranslations(<Component />)
  expect(screen.getByText('Translated text')).toBeInTheDocument()
})
```

### 3. Testing UI Components
Example from Button component tests (using `vi.fn`):
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { describe, it, expect, vi } from 'vitest'
import { Button } from '../Button'

describe('Button', () => {
  it('applies correct size classes', () => {
    const { rerender } = render(<Button size="sm">Small</Button>)
    expect(screen.getByRole('button', { name: 'Small' }))
      .toHaveClass('text-sm h-8 px-3') // jest-dom matcher

    rerender(<Button size="md">Medium</Button>)
    expect(screen.getByRole('button', { name: 'Medium' }))
      .toHaveClass('text-base h-10 px-4') // jest-dom matcher
  })

  it('handles click events', () => {
    const handleClick = vi.fn() // Use vi.fn()
    render(<Button onClick={handleClick}>Click me</Button>)

    fireEvent.click(screen.getByRole('button', { name: 'Click me' }))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LastBotInc/nextjs-supabase-ai-webapp](https://github.com/LastBotInc/nextjs-supabase-ai-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
