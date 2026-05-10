---
trigger: always_on
description: Comprehensive testing patterns for React Testing Library and Go testing in POS System
---


# 🧪 Testing Patterns & Best Practices

## 🎯 Testing Philosophy

### Testing Pyramid for POS System
```
    E2E Tests (Few)
    ↑ Full user workflows
    ↑ Critical business flows
    
  Integration Tests (Some)
  ↑ API + Database interactions
  ↑ Component + API integration
  
    Unit Tests (Many)
    ↑ Individual functions
    ↑ Component behavior
    ↑ Business logic validation
```

### Test Coverage Targets
- **Unit Tests:** 80%+ coverage for business logic
- **Integration Tests:** All API endpoints with database
- **E2E Tests:** Core user journeys (login → order → payment → kitchen)

## ⚛️ Frontend Testing Patterns (React Testing Library)

### Component Testing Setup
```typescript
// test-utils.tsx - Custom testing utilities
import { render, RenderOptions } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactElement } from 'react'
import { BrowserRouter } from '@tanstack/react-router'

// Create a test query client with no retries
const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,      // Don't retry on test failures
      gcTime: Infinity,  // Keep data in cache
    },
    mutations: {
      retry: false,
    },
  },
})

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  queryClient?: QueryClient
  initialEntries?: string[]
}

// Custom render with providers
export const renderWithProviders = (
  ui: ReactElement,
  {
    queryClient = createTestQueryClient(),
    initialEntries = ['/'],
    ...renderOptions
  }: CustomRenderOptions = {}
) => {
  const Wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter initialEntries={initialEntries}>
        {children}
      </BrowserRouter>
    </QueryClientProvider>
  )

  return { ...render(ui, { wrapper: Wrapper, ...renderOptions }), queryClient }
}

// Re-export everything
export * from '@testing-library/react'
```

### Component Testing Examples

#### 1. Testing POS Product Card
```typescript
// ProductCard.test.tsx
import { screen, userEvent } from '@testing-library/react'
import { renderWithProviders } from '../test-utils'
import { ProductCard } from '@/components/pos/ProductCard'
import { Product } from '@/types'

const mockProduct: Product = {
  id: '123',
  name: 'Cheeseburger',
  price: 12.99,
  category_id: 'burgers',
  is_available: true,
  description: 'Delicious beef burger',
  image_url: null,
}

describe('ProductCard', () => {
  const mockOnSelect = jest.fn()

  beforeEach(() => {
    mockOnSelect.mockClear()
  })

  it('displays product information correctly', () => {
    renderWithProviders(
      <ProductCard 
        product={mockProduct} 
        onSelect={mockOnSelect} 
        isSelected={false} 
      />
    )

    expect(screen.getByText('Cheeseburger')).toBeInTheDocument()
    expect(screen.getByText('$12.99')).toBeInTheDocument()
    expect(screen.getByText('Delicious beef burger')).toBeInTheDocument()
  })

  it('calls onSelect when clicked', async () => {
    const user = userEvent.setup()
    
    renderWithProviders(
      <ProductCard 
        product={mockProduct} 
        onSelect={mockOnSelect} 
        isSelected={false} 
      />
    )

    await user.click(screen.getByText('Cheeseburger'))
    expect(mockOnSelect).toHaveBeenCalledWith(mockProduct)
  })

  it('shows selected state correctly', () => {
    renderWithProviders(
      <ProductCard 
        product={mockProduct} 
        onSelect={mockOnSelect} 
        isSelected={true} 
      />
    )

    const card = screen.getByRole('button')
    expect(card).toHaveClass('ring-2', 'ring-primary')
  })

  it('disables unavailable products', () => {
    const unavailableProduct = { ...mockProduct, is_available: false }
    
    renderWithProviders(
      <ProductCard 
        product={unavailableProduct} 
        onSelect={mockOnSelect} 
        isSelected={false} 
      />
    )

    const card = screen.getByRole('button')
    expect(card).toBeDisabled()
    expect(screen.getByText('Unavailable')).toBeInTheDocument()
  })
})
```

#### 2. Testing Forms with React Hook Form
```typescript
// OrderForm.test.tsx
import { screen, userEvent, waitFor } from '@testing-library/react'
import { renderWithProviders } from '../test-utils'
import { OrderForm } from '@/components/forms/OrderForm'
import { CreateOrderRequest } from '@/types'

// Mock API client
jest.mock('@/api/client', () => ({
  createOrder: jest.fn(),
}))

describe('OrderForm', () => {
  const mockOnSubmit = jest.fn()
  const mockOnCancel = jest.fn()

  beforeEach(() => {
    mockOnSubmit.mockClear()
    mockOnCancel.mockClear()
  })

  it('renders form fields correctly', () => {
    renderWithProviders(
      <OrderForm onSubmit={mockOnSubmit} onCancel={mockOnCancel} />
    )

    expect(screen.getByLabelText(/order type/i)).toBeInTheDocument()
    expect(screen.getByLabelText(/customer name/i)).toBeInTheDocument()
    expect(screen.getByRole('button', { name: /create order/i })).toBeInTheDocument()
    expect(screen.getByRole('button', { name: /cancel/i })).toBeInTheDocument()
  })

  it('validates required fields', async () => {
    const user = userEvent.setup()
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
