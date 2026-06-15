---
trigger: always_on
description: Language-agnostic programming patterns: SOLID, design patterns, clean code, and architecture. Load when refactoring, designing abstractions, or reviewing structure — not for everyday syntax.
---


# Language-Agnostic Programming Patterns

Universal principles for structure, naming, architecture, and testing — applicable across all languages.

Load this rule when refactoring modules, designing abstractions, reviewing architecture, or choosing patterns. For day-to-day coding workflow (read-before-edit, CI discovery, minimal diff, verification), the always-on core **Code Discipline** section is canonical — do not duplicate it here. For the judgment layer — root-cause method, simplicity taste, test integrity — load `fable5-coding-craft` alongside this rule.

---

## Pattern Judgment (Read First)

Everything below is vocabulary, not a checklist. Frontier-quality code applies patterns *reactively* — when the code's actual pain demands them — never proactively because a situation pattern-matches a textbook example.

- Every pattern has a cost: indirection, a new concept for readers, more files to trace through. Apply one only when the pain it removes is already present, not predicted.
- The strongest signal for an abstraction is the **third occurrence** of real duplication with identical reasons to change. Two similar blocks that change for different reasons are not duplication — unifying them couples things that must stay free.
- SOLID violations matter when they cause observed friction (a class you cannot test, a switch you keep re-editing). A small concrete class that "violates SRP" but has never needed to change is fine code — leave it alone.
- The best architecture for most changes is the one the repo already has. Pattern fluency is mostly for *reading* existing designs and for naming the structure a refactor is already growing toward.
- When reviewing, flag pattern *overuse* with the same severity as pattern absence: a Strategy with one strategy, a Factory with one product, or an interface with one implementation is indirection without payoff.

---

## Change Discipline

- Solve the requested problem with the smallest vertical slice; expand only after it works.
- Prefer extending existing modules over creating parallel implementations.
- When adding a dependency, confirm the repo does not already solve the same need.
- Keep public surfaces stable unless the task explicitly requires a breaking change.
- Leave the codebase in a compilable, testable state after each meaningful step.

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m3-cursor-rules](https://github.com/madebyaris/advance-minimax-m3-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
