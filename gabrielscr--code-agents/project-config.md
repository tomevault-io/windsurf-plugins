---
trigger: always_on
description: This repository contains 86 specialized AI agents for full-stack development across multiple technologies. Use these instructions to leverage the agents effectively with GitHub Copilot.
---

# GitHub Copilot Instructions

This repository contains 86 specialized AI agents for full-stack development across multiple technologies. Use these instructions to leverage the agents effectively with GitHub Copilot.

## Repository Structure

```
agents/
├── frontend-ai-agents/          # 13 Frontend agents (React, Angular, Vue, etc.)
├── backend-ai-agents/           # 48 Backend agents (Python, Go, Java, Node.js, Ruby, Rust)
├── dotnet-ai-agents/backend/    # 11 .NET backend agents
└── flutter-ai-agent/            # 14 Flutter development agents
```

**Total: 86 Specialized Professional Agents**

## Using with GitHub Copilot

### Inline Comments for Context

When working on code, use comments to reference specific agents:

```typescript
// Following react-component-designer patterns for accessible components
// Using frontend-performance-optimizer best practices for Core Web Vitals
```

```python
# Following python-clean-architecture patterns for domain layer
# Using python-security-specialist best practices for input validation
```

```csharp
// Following dotnet-api-designer patterns for RESTful API
// Using dotnet-security-implementer best practices
```

```dart
// Following flutter-use-cases pattern with validation
// Using flutter-ui guidelines - UI only invokes Cubit
```

### Workspace Instructions

Add this to your `.vscode/settings.json`:

```json
{
  "github.copilot.advanced": {
    "instructionsFile": "agents/.github/copilot-instructions.md"
  }
}
```

## Agent Patterns by Language

### Frontend (React, Angular, Vue)

#### React Architecture (`react-architect`)
```typescript
// Pattern: Feature-based structure with Clean Architecture
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── api/
│   └── dashboard/
├── shared/
└── lib/
```

#### React Components (`react-component-designer`)
```typescript
// Pattern: Accessible compound components
interface SelectProps {
  value: string
  onChange: (value: string) => void
  children: React.ReactNode
}

export function Select({ value, onChange, children }: SelectProps) {
  return (
    <div role="combobox" aria-expanded={isOpen}>
      {children}
    </div>
  )
}
```

#### State Management (`react-state-manager`)
```typescript
// Pattern: Zustand for simple state
import { create } from 'zustand'

interface Store {
  count: number
  increment: () => void
}

export const useStore = create<Store>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}))

// Pattern: TanStack Query for server state
const { data, isLoading } = useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
})
```

#### Tailwind CSS (`tailwind-specialist`)
```typescript
// Pattern: Responsive design with custom variants
<div className="
  p-4 md:p-6 lg:p-8
  bg-white dark:bg-gray-800
  rounded-lg shadow-md
  hover:shadow-lg transition-shadow
">
  {/* Content */}
</div>
```

#### Performance Optimization (`frontend-performance-optimizer`)
```typescript
// Pattern: Virtual scrolling for large lists
const virtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 50,
  overscan: 5,
})

// Pattern: Lazy loading routes
const Dashboard = lazy(() => import('./pages/Dashboard'))
```

#### Testing (`frontend-tester`, `e2e-tester`)
```typescript
// Pattern: Testing Library for components
it('should submit form with valid data', async () => {
  render(<LoginForm />)

  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com')
  await userEvent.click(screen.getByRole('button', { name: /login/i }))

  expect(await screen.findByText(/welcome/i)).toBeInTheDocument()
})

// Pattern: Playwright for E2E
test('user can complete checkout flow', async ({ page }) => {
  await page.goto('/products')
  await page.click('text=Add to cart')
  await page.click('text=Checkout')
  await expect(page).toHaveURL(/checkout/)
})
```

### Backend (Python, Go, Java, Node.js, Ruby, Rust)

#### Python FastAPI (`python-fastapi-developer`, `python-clean-architecture`)
```python
# Pattern: Clean Architecture with dependency injection
class CreateUserUseCase:
    def __init__(self, user_repository: UserRepository):
        self._repository = user_repository

    async def execute(self, name: str, email: str) -> User:
        # Validation
        if not email or '@' not in email:
            raise ValidationError("Invalid email")

        # Business logic
        user = User(name=name, email=email)
        return await self._repository.create(user)

# Pattern: FastAPI endpoint with validation
@router.post("/users", response_model=UserResponse, status_code=201)
async def create_user(
    user_data: UserCreate,
    use_case: CreateUserUseCase = Depends(get_create_user_use_case)
):
    user = await use_case.execute(name=user_data.name, email=user_data.email)
    return UserResponse.from_entity(user)
```

#### Go REST API (`go-rest-api-developer`, `go-clean-architecture`)
```go
// Pattern: Repository pattern with interfaces
type UserRepository interface {
    Create(ctx context.Context, user *User) error

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrielscr/code-agents](https://github.com/gabrielscr/code-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
