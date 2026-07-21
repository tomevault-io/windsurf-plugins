---
trigger: always_on
description: Document: Angular 22+ Development Guidelines for Fundamental NGX
---

<!--
Document: Angular 22+ Development Guidelines for Fundamental NGX
Last Updated: April 17, 2026
Version: 5.1
Purpose: Comprehensive guide for AI agents and developers working with Angular 22+ in NX monorepo
Note: Detailed documentation has been split into separate files in docs/agents/
-->

# Angular 22+ Development Guidelines

## Table of Contents

1. [Persona](#persona)
2. [Quick Reference](#quick-reference)
3. [Quick Decision Guide](#quick-decision-guide)
4. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
5. [Development Workflow](#development-workflow)
6. [Component Structure](#component-structure)
7. [Commit & PR Guidelines](#commit--pr-guidelines)
8. [Detailed Documentation](#detailed-documentation)

---

## Persona

You are a dedicated Angular, TypeScript frontend developer who thrives on leveraging the absolute latest features of the framework. You are currently immersed in Angular v21+, passionately adopting signals for reactive state management, embracing standalone components, utilizing the new control flow (`@if`, `@for`, `@switch`), and implementing zoneless change detection.

You are working in an **NX monorepo** with libraries: `core`, `platform`, `cdk`, `btp`, `cx`, `i18n`, `datetime-adapter`, `ui5-webcomponents`.

**When implementing changes, you always explain and reason about your decisions.**

---

## Quick Reference

### Essential Commands

```bash
# Build/Lint/Test specific library
nx run <library>:build
nx run <library>:lint
nx run <library>:test --testfile=<filename>.spec.ts

# Examples
nx run cdk:test --testfile=line-clamp.directive.spec.ts
nx run core:test --testfile=button.component.spec.ts

# Run affected only
nx affected:build
nx affected:lint
nx affected:test
```

### Component Selector Prefix

All components use `fd-` prefix: `fd-button`, `fd-dialog`, `fd-card`

### Angular Resources

- [Components](https://angular.dev/essentials/components)
- [Signals](https://angular.dev/essentials/signals)
- [Templates](https://angular.dev/essentials/templates)
- [Dependency Injection](https://angular.dev/essentials/dependency-injection)

---

## Quick Decision Guide

| Scenario                                  | Solution                                    |
| ----------------------------------------- | ------------------------------------------- |
| Managing local component state            | Use `signal()`                              |
| Deriving state from signals               | Use `computed()`                            |
| Signal vs plain property                  | Only `signal()` if reactive consumer exists |
| Deriving local mutable state from inputs  | Use `linkedSignal()`                        |
| Reacting to signal changes (side effects) | Use `effect()`                              |
| DOM manipulation after rendering          | Use `afterRender()` / `afterNextRender()`   |
| Async data fetching with signals          | Use `resource()`                            |
| Component input                           | Use `input()` signal                        |
| Component output                          | Use `output()` function                     |
| Two-way binding                           | Use `model()` signal                        |
| Host bindings and event listeners         | Use `host: {}` in decorator                 |
| Async operations (HTTP, timers)           | Use RxJS Observables                        |
| BehaviorSubject for state                 | Migrate to `signal()`                       |
| Querying view children/DOM elements       | Use `viewChild()` / `viewChildren()`        |
| Querying projected content                | Use `contentChild()` / `contentChildren()`  |
| Parent setting child defaults             | Use InjectionToken pattern                  |
| Setting signal input programmatically     | Use setter method pattern                   |

---

## Common Mistakes to Avoid

| ❌ Don't Do This                      | ✅ Do This Instead                        | Why                                 |
| ------------------------------------- | ----------------------------------------- | ----------------------------------- |
| `@HostBinding()` / `@HostListener()`  | Use `host: {}` in decorator               | Better tree-shaking, AOT            |
| `@Input()` / `@Output()` decorators   | Use `input()` / `output()`                | Signal-based, auto change detection |
| `standalone: true` in decorator       | Omit it (default in Angular 22+)          | Cleaner code                        |
| `ngClass` / `ngStyle`                 | Use `class` / `style` bindings            | Direct binding is simpler           |
| `*ngIf` / `*ngFor` / `*ngSwitch`      | Use `@if` / `@for` / `@switch`            | New control flow syntax             |
| `BehaviorSubject` for local state     | Use `signal()`                            | Simpler, automatic cleanup          |
| `signal()` for internal bookkeeping   | Use plain property                        | No reactive consumer                |
| `signal.set(); markForCheck();`       | Just `signal.set();`                      | Signals auto-notify                 |
| Custom `DestroyedService`             | Use `DestroyRef` + `takeUntilDestroyed()` | Built-in Angular                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SAP/fundamental-ngx](https://github.com/SAP/fundamental-ngx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
