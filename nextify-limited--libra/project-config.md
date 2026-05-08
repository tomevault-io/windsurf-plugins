---
trigger: always_on
description: Guidelines for writing clean, maintainable, and human-readable code. Apply these rules when writing or reviewing code to ensure consistency and quality.
---


## Core Principles

**Why** Implementing consistent development philosophy across the project maximizes collaboration efficiency and reduces long-term maintenance costs.

**When** Check against these principles before writing or reviewing any code.

**Key Takeaways**
1. Follow project conventions (Biome / shadcn / Tailwind)
2. Keep it simple – avoid over-engineering
3. Scout principle – make code better with every change
4. Root cause analysis – use type system to prevent issues

### Example Summary
```typescript
// Single responsibility example
function validateEmail(email: string): boolean {
  return /^\S+@\S+\.\S+$/.test(email)
}
```

---

## Architecture Design Principles

**Why** Good architecture makes code sustainably evolve and easily extensible.

**When** Apply when designing new modules or refactoring existing ones.

**Key Takeaways**
1. Centralized, type-safe configuration management
2. Composition over inheritance
3. Concurrency handling: React 19 + Suspense
4. Moderate configuration / dependency injection / principle of least knowledge

### Example Summary
```typescript
// Dependency injection example
class OrderProcessor {
  constructor(private payment: PaymentService) {}
  async process(order: Order) {
    await this.payment.pay(order.total)
  }
}
```

---

## Code Readability Techniques

**Why** Readable code allows future you (or colleagues) to quickly understand business intent.

**When** Should always be considered during writing, refactoring, or code review.

**Key Takeaways**
1. Consistent naming and style
2. Use explanatory variables and positive conditions
3. Encapsulate boundary conditions & extract functions
4. Avoid logical dependencies and side effects

### Example Summary
```typescript
// Explanatory variable example
const isProUser = user.plan === 'pro'
const hasActiveSub = user.subscription === 'active'
if (isProUser && hasActiveSub) {
  /* ... */
}
```

---

## Naming Conventions

**Why** Clear, consistent naming minimizes team communication costs.

**When** When defining any variables, functions, files, or branch names.

**Key Takeaways**
1. Descriptive and searchable
2. Meaningful distinctions, not numeric suffixes
3. Use named constants instead of magic values
4. Avoid encoding prefixes (Hungarian notation)

### Example Summary
```typescript
// Constants example
const MAX_RETRY = 3
```

---

## Function Design Principles

**Why** Small, focused functions are easy to test and reuse.

**When** When implementing new features or evaluating existing function complexity.

**Key Takeaways**
1. Single responsibility & descriptive function names
2. Minimal parameter count (object parameter passing)
3. Avoid side effects and boolean flag parameters

### Example Summary
```typescript
interface CreateProjectParams {
  name: string
  type: ProjectType
}
function createProject({ name, type }: CreateProjectParams) {
  /* ... */
}
```

---

## Comment Best Practices

**Why** High-quality comments supplement, not replace, readable code.

**When** When explaining intent, complex algorithms, or potential risks.

**Key Takeaways**
1. Self-documenting code is better than comments
2. Avoid redundant/noise comments
3. Delete unused code instead of commenting it out
4. Use JSDoc to document interfaces and examples

### Example Summary
```typescript
/**
 * Calculate discounted price
 */
function calcDiscount(price: number, rate: number) {
  return price * (1 - rate)
}
```

---

## Code Organization Structure

**Why** Proper organization reduces module cross-dependencies and cognitive load.

**When** When creating new files or reorganizing existing logic.

**Key Takeaways**
1. Vertical separation of concepts & related code stays close
2. Declare variables/functions near their usage
3. Maintain line length and group with blank lines

### Example Summary
```typescript
// Good vertical separation - related concepts tightly organized
class UserService {
  // Private fields concentrated at the top
  private readonly userRepository: UserRepository
  private readonly emailService: EmailService

  constructor(userRepo: UserRepository, emailSvc: EmailService) {
    this.userRepository = userRepo
    this.emailService = emailSvc
  }

  // Public methods organized by call hierarchy
  async createUser(userData: CreateUserData): Promise<User> {
    const validatedData = this.validateUserData(userData)
    const user = await this.saveUser(validatedData)
    await this.sendWelcomeEmail(user)
    return user
  }

  // Private helper methods follow closely after the public methods that call them
  private validateUserData(data: CreateUserData): ValidatedUserData {
    if (!data.email || !data.name) {
      throw new Error('Missing required fields')
    }
    return { ...data, createdAt: new Date() }
  }

  private async saveUser(data: ValidatedUserData): Promise<User> {
    return await this.userRepository.save(data)
  }

  private async sendWelcomeEmail(user: User): Promise<void> {
    await this.emailService.sendWelcome(user.email, user.name)
  }
}

// Poor vertical separation - concepts scattered
class BadUserService {
  private userRepository: UserRepository


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nextify-limited/libra](https://github.com/nextify-limited/libra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
