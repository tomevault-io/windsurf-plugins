---
trigger: always_on
description: This document outlines our development standards and best practices. These guidelines emphasize pragmatic, maintainable code that solves real problems without unnecessary complexity.
---

# Development Standards

This document outlines our development standards and best practices. These guidelines emphasize pragmatic, maintainable code that solves real problems without unnecessary complexity.

## Working Ethos & Collaboration

### Critical Thinking
- Ask clarifying questions before jumping to conclusions
- Challenge assumptions—especially around requirements and edge cases
- Consider the broader context and implications of changes
- Think through failure modes and edge cases

### Communication
- Be explicit about assumptions and trade-offs
- Share knowledge and context with team members
- Document key decisions and their rationale
- Flag potential issues early

### Risk Management
- Flag risky, complex, or brittle implementations
- Identify potential failure points and dependencies
- Consider maintenance and operational implications
- Suggest alternatives when simpler approaches exist

### AI Assistant Collaboration
When working with AI assistants:
- Expect and encourage clarifying questions
- Provide context and constraints upfront
- Be specific about requirements and expectations
- Review and validate generated code and suggestions
- Consider AI suggestions as recommendations, not mandates

### Better Alternatives
Always be open to suggesting and considering better approaches when:
- A simpler solution exists
- A more maintainable approach is possible
- A more robust implementation can be achieved
- Technical debt can be avoided
- Performance can be significantly improved

## Core Principles

### Code Organization
- Follow Next.js conventions for project structure
  ```
  ✅ Good:
  src/
    app/           # Next.js app router pages and layouts
    components/    # Reusable UI components
    utils/         # Utility functions and helpers
    lib/          # Third-party library configurations
    types/        # TypeScript type definitions
    styles/       # Global styles and theme
    hooks/        # Custom React hooks
    tests/        # Test files matching source structure

  ❌ Avoid:
  features/
    feature-a/
    feature-b/
  ```

- Keep related code close together within the appropriate directory
- Use clear, intention-revealing names that reflect business concepts
- Place components close to where they are used when they are specific to a feature
- Use the app directory for routing and page-specific components

### Design Philosophy
- Prefer composition over inheritance
  - Build complex behavior by combining simple, focused pieces
  - Avoid deep inheritance hierarchies
  - Use interfaces to define contracts

- Apply Single Responsibility Principle pragmatically
  - Each unit should do one job well
  - "One job" means "one reason to change"
  - Balance between too granular and too coupled

- Think in terms of responsibilities and boundaries
  - Focus on what each module is responsible for
  - Define clear interfaces between modules
  - Hide implementation details

- Avoid premature abstraction
  ```
  // ✅ Good: Simple and direct
  function calculateTotal(items) {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  }

  // ❌ Avoid: Over-engineered
  class PriceCalculationStrategy {
    calculate(items) {
      throw new Error('Abstract method')
    }
  }
  class StandardPriceCalculator extends PriceCalculationStrategy {
    calculate(items) {
      return items.reduce((sum, item) => sum + item.price * item.quantity, 0)
    }
  }
  ```

### Code Structure

#### Functions and Components
- Write modular, testable functions
  - Pure functions where possible
  - Clear inputs and outputs
  - Minimal side effects
  - Easy to test in isolation

- Keep functions focused
  ```
  // ✅ Good: Single responsibility
  function validateEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }

  function notifyUser(user, message) {
    return sendEmail(user.email, message)
  }

  // ❌ Avoid: Mixed responsibilities
  function validateAndNotifyUser(email, message) {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      throw new Error('Invalid email')
    }
    return sendEmail(email, message)
  }
  ```

- Model real behaviors
  ```
  // ✅ Good: Models real-world concept
  const shoppingCart = {
    addItem(item, quantity) { /* ... */ },
    removeItem(itemId) { /* ... */ },
    updateQuantity(itemId, quantity) { /* ... */ },
    getTotal() { /* ... */ }
  }

  // ❌ Avoid: Arbitrary technical abstraction
  const cartManager = {
    executeOperation(operation) { /* ... */ },
    calculateMetrics() { /* ... */ }
  }
  ```

#### State Management
- Keep state as local as possible
- Lift state up only when necessary
- Choose appropriate state management tools based on:
  - Scale of application
  - Team experience
  - Performance requirements
  - Development workflow

## Testing Strategy

### Unit Tests
- Focus on pure logic
  - Utilities
  - Helpers
  - Business logic
  - State management

```
// ✅ Good: Testing pure business logic
describe('calculateOrderTotal', () => {
  it('correctly calculates total with multiple items', () => {
    const items = [
      { price: 10, quantity: 2 },
      { price: 15, quantity: 1 }
    ]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robbowley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
