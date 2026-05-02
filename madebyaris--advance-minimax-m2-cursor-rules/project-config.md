---
trigger: always_on
description: Language-agnostic programming patterns: SOLID principles, design patterns, clean code, architecture decisions applicable across all languages
---


# Language-Agnostic Programming Patterns

Universal principles and patterns applicable across all programming languages.

## Development Workflow

### CLI-First Principle

Before manually creating project scaffolding or dependency manifests, check if a CLI tool exists:

| Task | Check for CLI | Example |
|------|---------------|---------|
| New project | Scaffolding CLI | `npm create`, `cargo new`, `flutter create` |
| New component | Generator CLI | `npx shadcn-ui add`, `ng generate`, `rails generate` |
| New module | Framework CLI | `nest generate module` |
| Dependencies | Package manager | `npm install`, `pip install` |

### Verification-First Principle

Code is not done until it is verified at a level proportional to the change:

```
1. Write code
2. Build/compile: Does it compile?
3. Lint: Are there warnings?
4. Run: Does it execute?
5. Test: Does it work correctly?
```

### Version-Aware Development

When versions matter:

```text
1. Search the current version with the actual month and year
2. Check compatibility with the current stack
3. Then recommend installation or migration steps
```

---

## SOLID Principles

### Single Responsibility Principle (SRP)
A class/module should have only one reason to change.

**Apply when:**
- A function does multiple unrelated things
- A class has too many dependencies
- Changes in one area affect unrelated code

**Example Pattern:**
```
Bad:  UserService handles auth, profile, notifications, and billing
Good: AuthService, ProfileService, NotificationService, BillingService
```

### Open/Closed Principle (OCP)
Open for extension, closed for modification.

**Apply when:**
- Adding new features requires modifying existing code
- Switch statements grow with each new type
- Core logic changes for edge cases

**Example Pattern:**
```
Bad:  if type == "email" ... elif type == "sms" ... elif type == "push" ...
Good: NotificationStrategy interface with EmailStrategy, SMSStrategy, PushStrategy
```

### Liskov Substitution Principle (LSP)
Subtypes must be substitutable for their base types.

**Apply when:**
- Derived classes override behavior in unexpected ways
- Code checks for specific types before operating
- Inheritance creates illogical hierarchies

**Example Pattern:**
```
Bad:  Square extends Rectangle but can't independently set width/height
Good: Both Square and Rectangle implement Shape interface
```

### Interface Segregation Principle (ISP)
Clients shouldn't depend on interfaces they don't use.

**Apply when:**
- Classes implement methods they don't need
- Interfaces have too many methods
- Changes affect many unrelated implementations

**Example Pattern:**
```
Bad:  Animal interface with fly(), swim(), walk() - Penguin can't fly
Good: Flyable, Swimmable, Walkable interfaces
```

### Dependency Inversion Principle (DIP)
Depend on abstractions, not concretions.

**Apply when:**
- High-level modules import low-level modules directly
- Changing database/service requires code changes
- Testing requires real dependencies

**Example Pattern:**
```
Bad:  UserService directly imports MySQLDatabase
Good: UserService depends on DatabaseInterface, injected at runtime
```

## Common Design Patterns

### Creational Patterns

#### Factory Pattern
Use when object creation logic is complex or needs to be centralized.

```
When to use:
- Multiple similar objects with different configurations
- Object creation depends on runtime conditions
- Hiding complex initialization logic
```

#### Builder Pattern
Use for constructing complex objects step by step.

```
When to use:
- Objects with many optional parameters
- Complex configuration requirements
- Need for immutable objects with many fields
```

#### Singleton Pattern
Use sparingly for truly global, single-instance resources.

```
When to use:
- Configuration managers
- Connection pools
- Logger instances

Avoid when:
- It's just for convenience (use DI instead)
- Testing would be difficult
- Multiple instances might be needed later
```

### Structural Patterns

#### Adapter Pattern
Convert one interface to another that clients expect.

```
When to use:
- Integrating third-party libraries
- Working with legacy code
- Unifying different data sources
```

#### Decorator Pattern
Add behavior to objects dynamically.

```
When to use:
- Adding features without subclassing
- Composable behaviors
- Middleware-like patterns
```

#### Facade Pattern
Provide a simplified interface to a complex subsystem.

```
When to use:
- Simplifying complex library usage
- Creating API boundaries
- Reducing coupling between layers
```

### Behavioral Patterns

#### Strategy Pattern
Define a family of interchangeable algorithms.

```
When to use:
- Multiple algorithms for the same task
- Runtime algorithm selection
- Avoiding complex conditionals
```

#### Observer Pattern
Notify dependents of state changes.

```
When to use:
- Event-driven systems
- Pub/sub messaging
- Reactive data flows
```

#### Command Pattern
Encapsulate requests as objects.

```
When to use:
- Undo/redo functionality
- Queueing operations
- Macro recording
```

## Clean Code Principles

### Naming Conventions

#### Variables and Functions
- Use intention-revealing names

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
