---
trigger: always_on
description: - Rules are tools, not goals in themselves
---


# Coding Standards

## Philosophy: Pragmatic Approach

- Rules are tools, not goals in themselves
- Readability > Conciseness
- Simple code is better than "perfect" code that follows all rules
- Don't force rules

## KISS (Keep It Simple, Stupid)

- **Simplicity over complexity** - choose simpler solutions
- **Don't overdo abstraction** - sometimes simple code is better
- **Avoid premature optimization** - working code first, then optimization
- **Readability > Performance** (in most cases)
- **The simplest solution that works** is the best

## SOLID (apply pragmatically)

- **SRP**: One class = one responsibility
- **OCP**: Open for extension, closed for modification
- **LSP**: Subclasses must be substitutable
- **ISP**: Specific interfaces, not general ones
- **DIP**: Depend on abstractions, not concrete implementations

## DRY (Don't Repeat Yourself)

- Refactor duplication when it occurs in 3+ places
- Don't overdo abstraction - sometimes duplication is more readable
- Refactor only when it makes practical sense

## CUPID

- **C**omposable - easy to compose
- **U**nix philosophy - does one thing well
- **P**redictable - predictable
- **I**diomatic - follows Laravel conventions
- **D**omain-based - domain-based

## Architecture Choice

### Principle: Start Simple, Scale When Needed

**Start with a simple solution, add complexity only when justified.**

### Simple Solutions (Transaction Script)

**Use when:**
- ✅ Small to medium project (MVP, prototypes)
- ✅ Simple business logic (CRUD, simple operations)
- ✅ Small team (1-3 people)
- ✅ Fast development and delivery
- ✅ Low risk of requirement changes
- ✅ Simple domain (doesn't require complex business logic)

**Examples:**
- Laravel Controllers with direct access to Models
- Simple Services with business logic
- Repository pattern (if needed)
- Minimal abstraction

**Advantages:**
- Fast development
- Easy to understand
- Low maintenance costs (initially)
- Fast iteration

### Advanced Architectures (DDD, Hexagonal, Clean Architecture)

**Use when:**
- ✅ Large, complex project
- ✅ Complex business logic (many rules, states)
- ✅ Multiple teams working in parallel
- ✅ Long project lifespan (5+ years)
- ✅ High risk of requirement changes
- ✅ Complex domain (domain experts, rich domain model)
- ✅ High testability and isolation required
- ✅ Need for multiple ports (API, CLI, Web, Events)

**Examples:**
- Domain-Driven Design (DDD) - Entities, Value Objects, Domain Services
- Hexagonal Architecture - Ports & Adapters
- Clean Architecture - layers with dependency rules
- CQRS + Event Sourcing (when needed)

**Advantages:**
- High testability
- Better separation of responsibilities
- Easier to add new features
- Framework independence
- Long-term maintainability

**Disadvantages:**
- More code (boilerplate)
- Slower initial development
- More abstraction (can be harder to understand)
- Higher initial costs

### Migration: Simple → Advanced

**When to refactor to more advanced architecture:**

1. ✅ Code becomes hard to maintain
2. ✅ Hard to add new features without changing existing ones
3. ✅ Tests are hard to write
4. ✅ Lots of business logic duplication
5. ✅ Team grows and needs better organization
6. ✅ Domain becomes more complex
7. ✅ Framework independence required

**When NOT to refactor:**
- ❌ "For the principle" - without a concrete problem
- ❌ When code works well and is easy to maintain
- ❌ When project is small and simple
- ❌ When there's no time/resources for refactoring

### Recommendations for MovieMind API

**Current state:**
- Simple architecture with Services, Repositories, Controllers
- Event-Driven for asynchronous operations (Jobs)
- Laravel conventions

**When to consider DDD/Hexagonal:**
- When business logic becomes very complex
- When need for many different ports (Web, CLI, Queue, Events)
- When team grows and needs better separation
- When need for independence from Laravel (e.g., shared kernel)

**Principle:**
**Start simple, refactor when concrete problems arise, not "just in case".**

## GRASP (General Responsibility Assignment Software Patterns)

GRASP patterns help assign responsibilities in object-oriented design:

- **Information Expert** - assign responsibility to class with information needed for execution
- **Creator** - assign creation responsibility to class that uses/contains object
- **Controller** - assign handling responsibility to class representing system/facade
- **Low Coupling** - minimize dependencies between classes
- **High Cohesion** - keep related functionality together
- **Polymorphism** - use polymorphism for behavioral variations
- **Pure Fabrication** - create classes that don't represent domain concepts (e.g., Repository)
- **Indirection** - use intermediate objects to loosen dependencies
- **Protected Variations** - protect against variations through encapsulation

## YAGNI (You Aren't Gonna Need It)

- **Don't add functionality until needed** - avoid premature abstraction
- **Solve today's problems, not tomorrow's** - focus on current requirements
- **Refactor when needed** - add complexity only when there's a concrete need

## Code Smells (fix when they hinder work)

### Structural Smells
- **God Class/Method** - class/method does too much → split into smaller classes/methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
